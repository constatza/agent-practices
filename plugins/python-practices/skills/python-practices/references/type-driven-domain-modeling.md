# Python Type-Driven Domain Modeling

## Goal

Use Python's type system, immutable structures, runtime validation, and static analysis to make invalid states hard to represent. Python cannot provide Rust-level guarantees, but it should encode domain invariants structurally whenever practical.

Prefer the simplest mechanism that prevents a plausible error. Do not add type-level complexity for its own sake.

## Choose the Representation

When code contains a `bool`, string status, `T | None`, `dict[str, Any]`, fields that must agree, repeated validation, a runtime “not initialized” check, or a “call X before Y” requirement, consider whether one of these representations would encode the invariant more directly:

- `A | B` union.
- `NewType`.
- `Literal` or `StrEnum`.
- Frozen dataclass.
- Validated value object.
- State-specific class.
- Explicit transition method.

## Model Alternatives as a Union

Avoid a flag coupled to optional state-specific data:

```python
from dataclasses import dataclass


@dataclass(frozen=True, slots=True)
class Payment:
    is_paid: bool
    transaction_id: str | None
```

Prefer separate variants that remove contradictory combinations:

```python
from dataclasses import dataclass


@dataclass(frozen=True, slots=True)
class PendingPayment:
    pass


@dataclass(frozen=True, slots=True)
class PaidPayment:
    transaction_id: str


Payment = PendingPayment | PaidPayment
```

Handle a closed union exhaustively:

```python
from typing import assert_never


def describe_payment(payment: Payment) -> str:
    match payment:
        case PendingPayment():
            return "pending"
        case PaidPayment(transaction_id=transaction_id):
            return f"paid: {transaction_id}"
        case unreachable:
            assert_never(unreachable)
```

Adding a union member should cause the static checker to expose affected matches.

## Use `StrEnum` Only for Symbolic Values

Use `StrEnum` instead of free-form strings for a closed symbolic set:

```python
from enum import StrEnum


class OrderStatus(StrEnum):
    DRAFT = "draft"
    SUBMITTED = "submitted"
    PAID = "paid"
```

Do not use an enum alone when each state requires different data. This still permits inconsistent combinations:

```python
from dataclasses import dataclass


@dataclass(frozen=True, slots=True)
class Order:
    status: OrderStatus
    payment_id: str | None
```

## Use State-Specific Dataclasses

Give each lifecycle state only its valid data and operations:

```python
from dataclasses import dataclass


@dataclass(frozen=True, slots=True)
class DraftOrder:
    items: tuple[str, ...]

    def submit(self) -> "SubmittedOrder":
        return SubmittedOrder(items=self.items)


@dataclass(frozen=True, slots=True)
class SubmittedOrder:
    items: tuple[str, ...]

    def pay(self, payment_id: str) -> "PaidOrder":
        return PaidOrder(items=self.items, payment_id=payment_id)


@dataclass(frozen=True, slots=True)
class PaidOrder:
    items: tuple[str, ...]
    payment_id: str
```

The API now expresses the valid lifecycle:

```text
DraftOrder
    ↓ submit()
SubmittedOrder
    ↓ pay()
PaidOrder
```

Prefer immutable transitions such as:

```python
submitted = draft.submit()
paid = submitted.pay(payment_id)
```

over field mutation that can leave a partially valid object.

## Choose `NewType` or a Value Object

Use `NewType` when no runtime validation is needed, zero-cost semantic distinction is sufficient, and accidental mixing is plausible:

```python
from typing import NewType


AccountId = NewType("AccountId", int)
AmountCents = NewType("AmountCents", int)


def transfer(
    from_account: AccountId,
    to_account: AccountId,
    amount: AmountCents,
) -> None:
    ...
```

Use an immutable value object when validation or behavior belongs to the value:

```python
from dataclasses import dataclass


@dataclass(frozen=True, slots=True)
class LearningRate:
    value: float

    def __post_init__(self) -> None:
        if not 0.0 < self.value <= 1.0:
            raise ValueError("Learning rate must satisfy 0 < value <= 1.")


def train(learning_rate: LearningRate) -> None:
    ...
```

Once constructed, downstream code should trust the value object's invariant. A frozen dataclass provides a shallow frozen shell: mutable values stored in its fields can still change. Do not expose such values, or copy/wrap them, when their mutation could violate the invariant.

## Validate External Data at the Boundary

Use the project's existing boundary-validation mechanism for untrusted data. For complex structured input, Pydantic v2 is a good choice when the project already uses it or the dependency is justified. Enable strict validation when coercion would conceal invalid input, then convert validated input into domain types:

```python
from typing import Annotated

from pydantic import BaseModel, ConfigDict, Field


LearningRateInput = Annotated[float, Field(gt=0.0, le=1.0)]


class TrainingConfigInput(BaseModel):
    model_config = ConfigDict(strict=True)
    learning_rate: LearningRateInput
```

```text
JSON / API / CLI / config / database
    ↓
boundary validation
    ↓
domain conversion
    ↓
validated domain types
    ↓
business logic
```

Boundary validation and domain modeling remain separate. A Pydantic input model does not automatically become the domain model.

## Make Lifecycle Operations State-Specific

Avoid a single mutable or partially optional lifecycle object:

```python
from dataclasses import dataclass

from torch import Tensor


@dataclass
class Model:
    fitted: bool
    weights: Tensor | None
```

Prefer types exposing only the operations valid in each state:

```python
from dataclasses import dataclass

from torch import Tensor


@dataclass(frozen=True, slots=True)
class UnfittedModel:
    hidden_dim: int

    def fit(self, x: Tensor, y: Tensor) -> "FittedModel":
        weights = train_weights(x=x, y=y, hidden_dim=self.hidden_dim)
        return FittedModel(weights=weights)


@dataclass(frozen=True, slots=True)
class FittedModel:
    weights: Tensor

    def predict(self, x: Tensor) -> Tensor:
        return x @ self.weights
```

The type checker should reject `UnfittedModel(...).predict(x)` because that operation does not exist in the unfitted state. Here, `frozen=True` prevents rebinding `weights`, but it does not make the tensor's contents immutable; copy or encapsulate the tensor if the model's invariant depends on those contents remaining unchanged.

## Represent Domain Outcomes Explicitly

Avoid outcome records whose fields must agree:

```python
from dataclasses import dataclass


@dataclass
class TrainingResult:
    succeeded: bool
    model: FittedModel | None
    reason: str | None
```

Prefer an explicit union:

```python
from dataclasses import dataclass


@dataclass(frozen=True, slots=True)
class TrainingSuccess:
    model: FittedModel


@dataclass(frozen=True, slots=True)
class TrainingFailure:
    reason: str


TrainingResult = TrainingSuccess | TrainingFailure
```

Handle it exhaustively:

```python
from typing import assert_never


def handle_result(result: TrainingResult) -> None:
    match result:
        case TrainingSuccess(model=model):
            save_model(model)
        case TrainingFailure(reason=reason):
            log_failure(reason)
        case unreachable:
            assert_never(unreachable)
```

Use this pattern when success and failure are domain outcomes represented as data. Continue to use exceptions for ordinary Python error transport.

## Distinguish Scientific and ML Roles

Use separate types when dataset roles constrain valid operations:

```python
from dataclasses import dataclass

from torch import Tensor


@dataclass(frozen=True, slots=True)
class TrainingSet:
    x: Tensor
    y: Tensor


@dataclass(frozen=True, slots=True)
class ValidationSet:
    x: Tensor
    y: Tensor


@dataclass(frozen=True, slots=True)
class TestSet:
    x: Tensor
    y: Tensor


def fit(model: UnfittedModel, dataset: TrainingSet) -> FittedModel:
    ...
```

A strict static checker should reject a `TestSet` passed to `fit`.

## Final Rule

Do not make Python imitate Rust mechanically. Apply the same modeling principle: encode domain truths in structures and types so incorrect code is harder to write and easier for static tooling to reject.
