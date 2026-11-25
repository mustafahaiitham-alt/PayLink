# PayLink — Requirements Document

## 1. Problem Summary

PayLink is a bill-payment and digital wallet backend system operating in Egypt. It allows customers to pay electricity, water, mobile top-up, and internet bills using money stored in their digital wallets.

Customers can use PayLink through a mobile application or through agents at small shops using PayLink terminals.

The system connects to different billers, calculates service fees, processes payments, manages wallet balances, and sends payment notifications.

The main challenge is handling money correctly and securely. The system must prevent duplicate charges, avoid negative wallet balances, handle biller failures and timeouts, and keep accurate records of every financial transaction.

PayLink aims to provide a reliable and extensible backend that can support different billers with different integration formats.

## 2. Stakeholders

| Stakeholder | Role / Interest |
|---|---|
| Customers | Register, manage wallets, view bills, pay bills, and receive notifications. |
| Agents | Pay bills on behalf of customers and earn commissions. |
| Administrators | Manage billers, services, fees, limits, and system configuration. |
| Support Employees | Search payments, investigate problems, and manage blocked accounts. |
| Billers | Receive payments and provide bill information or payment confirmations. |
| PayLink Management | Monitor payments, fees, commissions, and system performance. |
| Development Team | Build, test, maintain, and extend the backend system. |

## 3. Functional Requirements — User Stories

Each user story uses the format:
- Given: The initial situation.
- When: The user performs an action.
- Then: The expected system result.

### FR-1: Customer Registration
**Priority: Must**

As a customer, I want to register an account so that I can use PayLink services.

- Given I am not registered.
- When I submit valid registration information.
- Then the system creates my account with basic verification level.

### FR-2: Upgrade Verification Level
**Priority: Should**{}

As a customer, I want to upgrade my verification level so that I can have higher payment limits.

- Given I have a basic account.
- When my verification upgrade is approved.
- Then my account becomes verified and my daily payment limit increases to 30,000 EGP.

### FR-3: View Wallet and Transaction History
**Priority: Must**

As a customer, I want to view my wallet balance and transaction history so that I can track my money.

- Given I am logged in.
- When I request my wallet information.
- Then the system displays my available balance, held balance, and transaction history.

### FR-4: Top Up Wallet
**Priority: Must**

As a customer, I want to add money to my wallet so that I can pay bills.

- Given I have a registered wallet.
- When I complete a simulated card top-up or an agent cash top-up.
- Then the system increases my wallet balance and records the transaction.

### FR-5: Record Every Balance Change
**Priority: Must**

As a system administrator, I want every wallet balance change to be recorded in a ledger so that financial records remain accurate.

- Given a wallet balance changes.
- When a deposit, payment, hold, or refund occurs.
- Then the system creates a ledger entry and calculates the balance from ledger entries instead of directly overwriting it.

### FR-6: Manage Billers and Services
**Priority: Must**

As an administrator, I want to add, edit, and disable billers and their services so that the available services remain up to date.

- Given I have administrator permissions.
- When I add, edit, or disable a biller or service.
- Then the system saves the changes and updates the available catalog.

### FR-7: View Billers and Required Fields
**Priority: Must**

As a customer, I want to see available billers and their required input fields so that I can provide the correct information.

- Given billers and services are available.
- When I open the biller catalog.
- Then the system displays active billers, their services, and required fields such as meter number or phone number.

### FR-8: Bill Inquiry
**Priority: Must**

As a customer, I want to inquire about my bill before paying so that I know the amount I owe.

- Given I select a biller and enter valid required information.
- When I request a bill inquiry.
- Then the system displays the bill amount, service fee, and total amount to pay.

### FR-9: Pay a Bill
**Priority: Must**

As a customer, I want to pay a bill using my wallet so that I can complete payments without visiting the biller's office.

- Given I have sufficient available wallet funds and the payment passes the required checks.
- When I confirm the bill payment.
- Then the system processes the payment through the payment lifecycle and sends it to the selected biller.

### FR-10: Prevent Duplicate Requests
**Priority: Must**

As a customer, I want repeated payment requests to return the original result so that I am not charged twice.

- Given I already submitted a payment request with an idempotency key.
- When I submit the same request again using the same key.
- Then the system returns the original payment result without creating another payment or charge.

### FR-11: Follow Payment Lifecycle
**Priority: Must**

As a system, I want payments to follow valid lifecycle transitions so that payment statuses remain consistent.

- Given a payment is in a specific state.
- When an event occurs.
- Then the system allows only the transitions defined in the payment lifecycle and rejects invalid transitions.

### FR-12: Hold and Release Funds
**Priority: Must**

As a customer, I want payment funds to be held before the biller is called and released if the payment fails so that my wallet remains accurate.

- Given I have enough available funds.
- When a payment passes the required checks.
- Then the system moves the payment amount from available balance to held balance before calling the biller.
- Given the biller refuses the payment.
- When the payment becomes Failed.
- Then the system releases the held funds and marks the payment as Refunded.

### FR-13: Handle Payment Timeouts
**Priority: Must**

As a customer, I want the system to handle biller timeouts safely so that I know when my payment is still being processed.

- Given a payment has been sent to a live biller.
- When the biller does not respond within 10 seconds.
- Then the payment becomes Unknown, the funds remain held, and the system follows the retry schedule of 1, 5, and 15 minutes before waiting for reconciliation.

### FR-14: Handle Asynchronous Callbacks
**Priority: Must**

As a system, I want to receive asynchronous callbacks from TopUpNet so that I can update mobile top-up payment results.

- Given a TopUpNet payment has been accepted.
- When TopUpNet sends a success or failure callback.
- Then the system updates the payment according to its current state.
- When the same callback is received more than once.
- Then duplicate callbacks are ignored and logged.

### FR-15: Support Different Billers
**Priority: Must**

As a customer, I want to pay through different billers so that I can use PayLink for multiple services.

- Given the system has configured billers.
- When I select a biller and use its service.
- Then the system supports at least ElectroCo, AquaCo, and TopUpNet using their different integration formats.

### FR-16: Calculate Service Fees
**Priority: Must**

As a customer, I want service fees to be calculated correctly so that I know the exact total before payment.

- Given I select a biller and enter a payment amount.
- When the system calculates the service fee.
- Then it applies the configured fee rules:
  - Electricity and water: fixed 2.50 EGP.
  - Mobile top-up: 1.5% of the amount, minimum 1 EGP.
  - Internet: 2 EGP up to 200 EGP, and 4 EGP above 200 EGP.
  - During a configured promotion period, the fee is zero for the selected biller.

### FR-17: Agent Payments and Commission
**Priority: Should**

As an agent, I want to pay bills on behalf of customers so that I can provide PayLink services and earn commissions.

- Given an agent is authorized and a customer payment is completed through the agent.
- When the payment is successfully completed.
- Then the system credits the agent's wallet with 40% of the service fee.

### FR-18: Enforce Payment Limits
**Priority: Must**

As a customer, I want the system to check payment limits so that payments outside the allowed limits are rejected.

- Given I submit a payment request.
- When the payment amount is below 5 EGP, above 10,000 EGP, or exceeds my daily limit.
- Then the system rejects the payment and provides a clear reason.
- Basic accounts have a daily limit of 5,000 EGP.
- Verified accounts have a daily limit of 30,000 EGP.

### FR-19: Prevent Excessive Payment Activity
**Priority: Should**

As a system, I want to detect excessive payment activity so that suspicious payment behavior can be temporarily blocked.

- Given an account makes payments.
- When it makes more than 5 payments within 60 seconds.
- Then the system blocks the account for 10 minutes.

### FR-20: Search Payment History
**Priority: Must**

As a support employee, I want to search for a payment using its reference so that I can investigate customer issues.

- Given I have support permissions and a payment reference.
- When I search for that reference.
- Then the system displays the payment's full history.

### FR-21: Block and Unblock Accounts
**Priority: Should**

As a support employee, I want to block or unblock an account so that I can help manage problematic accounts.

- Given I have support permissions.
- When I block or unblock an account.
- Then the system updates the account status and records the administrative action.

### FR-22: Send Payment Notifications
**Priority: Must**

As a customer, I want to receive payment notifications so that I know whether my payment succeeded or failed.

- Given a payment reaches Completed or Failed.
- When the payment status changes.
- Then the system sends an SMS and a push notification to the customer.

### FR-23: Retry Failed Notifications
**Priority: Should**

As a system, I want to retry failed notifications so that temporary provider failures do not prevent notification delivery.

- Given an SMS or email notification fails.
- When the system performs its retry process.
- Then it retries the notification up to 3 times and marks it as undelivered if all attempts fail.

### FR-24: Update Fees and Limits
**Priority: Should**

As an administrator, I want to edit fee and limit values without redeploying the system so that business rules can be updated easily.

- Given I have administrator permissions.
- When I update a fee or limit value.
- Then the system saves the new configuration and applies it without requiring redeployment.

### FR-25: Audit Financial and Administrative Actions
**Priority: Must**

As a system administrator, I want money movements and administrative actions to be recorded so that important operations can be tracked.

- Given a money movement or administrative action occurs.
- When the action is completed.
- Then the system writes an audit record containing who performed it, when it happened, and why.

### FR-26: Daily Reconciliation Report
**Priority: Should**

As a support employee, I want a daily reconciliation report so that I can identify differences between PayLink records and biller settlement files.

- Given a biller settlement file is available.
- When the reconciliation process runs.
- Then the system compares the settlement file with PayLink payment records and produces a report highlighting differences.

### FR-27: Request a Refund for a Completed Payment
**Priority: Could**

As a customer, I want to request a refund for a completed payment so that I can ask PayLink to return money when necessary.

- Given I have a completed payment.
- When I submit a refund request.
- Then the system records the request for support approval.

### FR-28: View Daily Business Totals
**Priority: Could**

As an administrator, I want to view daily business totals so that I can monitor PayLink operations.

- Given payment and financial records exist.
- When I open the daily totals report.
- Then the system displays the number of payments, fees earned, commissions paid, and failures per biller.

## 4. Non-Functional Requirements (NFRs)

The following measurable quality requirements are based on the project brief.

| ID | Quality Attribute | Measurable Requirement |
|---|---|---|
| NFR-1 | Correctness | The system must produce zero duplicate charges and zero negative wallet balances in duplicate-request and concurrency tests. |
| NFR-2 | Performance | The system must sustain 100 payments per second for 1 minute against mock billers. |
| NFR-3 | Resilience | An outage of one biller must not stop payment processing for other billers. |
| NFR-4 | Modifiability | Adding a new biller must require changes only to the Biller Integration module and configuration; existing modules must remain unchanged. |
| NFR-5 | Auditability | 100% of money movements must have an audit record containing who, when, and why. |
| NFR-6 | Testability | Domain logic must achieve at least 70% line coverage, measured in the CI pipeline. |

## 5. Priorities

The priorities below follow the original functional requirements in the project brief.

- **Must:** Essential requirements that must be implemented.
- **Should:** Important requirements that should be implemented, but are not marked Must in the brief.
- **Could:** Optional requirements that may be implemented if time allows.

### Must
FR-1, FR-3, FR-4, FR-5, FR-6, FR-7, FR-8, FR-9, FR-10, FR-11, FR-12, FR-13, FR-14, FR-15, FR-16, FR-18, FR-20, FR-22, FR-25.

### Should
FR-2, FR-17, FR-19, FR-21, FR-23, FR-24, FR-26.

### Could
FR-27, FR-28.

## 6. Ambiguities, Questions, and Decisions

The following points were reviewed while translating the project brief into testable requirements.

| Question / Ambiguity | Decision |
|---|---|
| What happens if the mobile app sends the same payment request twice? | The system uses an idempotency key. Repeated requests return the original result and must not create another charge. |
| What happens if a biller does not respond? | After the 10-second timeout, the payment becomes Unknown and the funds remain held. |
| Should an Unknown payment be refunded immediately? | No. It must not be refunded automatically until the system is sure the biller did not receive the payment. |
| What happens when a biller refuses a payment? | The payment becomes Failed, the hold is released, and the payment becomes Refunded. |
| What happens if two payments arrive at the same time and the wallet can cover only one? | Only one payment succeeds; the other is rejected. The wallet balance must never become negative. |
| What happens if TopUpNet sends the same callback more than once? | The payment is completed only once. Duplicate callbacks are ignored and logged. |
| What happens if the SMS provider is unavailable? | The payment still completes. The notification is retried up to 3 times and then marked undelivered. |
| How should different biller protocols be handled? | Each biller has its own integration adapter so that its protocol and data format are hidden from the rest of the system. |
| What happens if a settlement file disagrees with PayLink's payment status? | The difference is flagged in the reconciliation report for support review. |
| Can the system support adding another biller later? | A new biller should require only a new integration and configuration, without changing existing modules. |
| What happens when an agent is involved in a successful payment? | The agent receives 40% of the service fee in the agent wallet. |
| What happens if a customer requests a refund for a Completed payment? | The system records the request for support approval. The exact processing and state transition for this case need confirmation. |

## 7. Business Rules

The following values are fixed by the project brief and should be stored in configuration rather than hard-coded.

| Business Rule | Value |
|---|---|
| Basic account daily payment limit | 5,000 EGP |
| Verified account daily payment limit | 30,000 EGP |
| Minimum single payment | 5 EGP |
| Maximum single payment | 10,000 EGP |
| Electricity and water fee | Fixed 2.50 EGP |
| Mobile top-up fee | 1.5%, minimum 1 EGP |
| Internet fee | 2 EGP up to 200 EGP; 4 EGP above 200 EGP |
| Promotion | Zero fee for one configured biller during configured start and end dates |
| Agent commission | 40% of service fee |
| Velocity rule | More than 5 payments in 60 seconds; block for 10 minutes |
| Live biller timeout | 10 seconds |
| Unknown payment retries | After 1, 5, and 15 minutes; then wait for reconciliation |
| Notification retries | 3 attempts |
| Money precision | Exact decimal types; never use floating-point types |

## 8. Technical Constraints

- The programming language and framework must be selected by the team and declared in Phase 1.
- Supported choices include Java Spring Boot, .NET, Node NestJS, or Python FastAPI.
- The database must be relational and contain at least 10 tables.
- Database schema migrations must be stored in the repository.
- The backend must provide at least 20 REST endpoints documented with OpenAPI.
- The system must support at least one asynchronous flow.
- Phase 1 architecture must be a layered or hexagonal modular monolith.
- Each module must have its own package or service, tests, and clear public interface.
- The project must use Docker and Docker Compose.

## 9. Scope of the First Release

The required core modules are:

1. Accounts and Wallet
2. Biller Catalog
3. Payment Processing
4. Biller Integration
5. Fees and Commissions
6. Notifications

Fraud Checks and Reconciliation are listed as extra modules in the project brief. However, their related business rules and hard cases still need to be considered where the brief explicitly requires them.

## 10. Open Questions Requiring Team Confirmation

The following details are not fully specified in the brief and should be confirmed by the team or instructor before implementation:

1. What exact customer registration fields and verification documents are required?
2. What exact fields are required for simulated card top-ups and agent cash top-ups?
3. What are the exact rules and permissions for support approval of refunds on Completed payments?
4. What authentication and authorization mechanism will be used for customers, agents, administrators, and support employees?
5. What exact data format and endpoint contracts will be used between PayLink and each mock biller?

These questions should be discussed and documented before the related features are implemented.
