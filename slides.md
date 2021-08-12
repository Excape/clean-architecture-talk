---
# try also 'default' to start simple
theme: default
layout: fact
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
class: text-center
lineNumbers: true
# some information about the slides, markdown enabled
info: |
  ## Clean Architecture in YNAB splitter
---

## Clean Architecture by Example
### Robin Suter

<img src="/clean-arch-book.jpeg" class="book" />
<img src="/ynab-logo.png" class="ynab-logo" />
<img src="/ynab-splitter-screenshot.png" class="ynab-splitter" />
<img src="https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg" class="diagram" />

<style>
  .book {
    @apply absolute bottom-0 left-0 ml-15 mb-15;
    transform: rotate(5deg);
    height: 20vh;
  }
  .ynab-logo {
    @apply absolute top-0 right-0 mt-10 mr-20;
    transform: rotate(3deg);
    height: 15vh;
  }
  .ynab-splitter {
    @apply absolute bottom-0 right-0 mr-15 mb-15;
    transform: rotate(-7deg);
    height: 23vh;
  }

  .diagram {
    @apply absolute top-0 left-0 mt-10 ml-10;
    transform: rotate(-5deg);
    height: 15vh;
  }
  h2 {
    @apply text-4xl;
  }
</style>

---

# Agenda

- Introduction to Clean Architecture
- Implementing Clean Architecture
  - Case Study "YNAB-Splitter"
  - Code Demo
  - Challenges / Discussion

---

# Clean Architecture
- Combination of different architectural styles (Hexagonal Architecture, DCI, BCE)
- Core Principles [^1]
  - Independent of frameworks
  - Testable
  - Independent of the _UI_
  - Independent of the _database_
  - Independent of any _external agency_

[^1]: [From Uncle Bob's Blog](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)


---

## The Dependency Rule

> Source code dependencies must point only inward, toward higher-level dependencies

<img src="https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg"/>


<style>
  img {
    height: 40vh;
  }
</style>

---

## The Framework is a detail
- Your core business logic should be _independent_ of any
  - Dependency Injection Framework (e.g. Spring)
  - Database Mapper
  - REST Framework
  - etc.

<img src="https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg"/>

<style>
  img {
    height: 40vh;
    @apply absolute bottom-0 right-0 pr-16 pb-16;
  }
</style>

---

# Implementing Clean Architecture

- Clean Architecture gives us principles, but no clear guidance for implementation
- We are looking at one possible approach in the context of a web application
- Thanks to Christoph Buchendorfer / Oliver Zihler

---

## Case Study: YNAB Splitter

- YNAB ("You need a Budget")[^1] is a personal financing and budgeting tool
- All purchases and transactions are recorded and money is budgeted into categories
  - All transactions have to be categorized

<img src="/ynab-logo.png" class="logo" />

<div class="flex flex-row justify-around mt-5">
  <img src="/ynab-website.png" class="website" />
  <img src="/ynab_desktop.png" class="desktop" />
</div>

<style>
  .logo {
    height: 12vh;
    @apply absolute top-0 right-0 mt-10 mr-10;
  }
  .desktop {
    height: 25vh;
  }
  .website {
    height: 25vh;
  }
</style>


[^1]: [YNAB website](https://www.youneedabudget.com/)

---

## The Problem

- My wife and I are using separate YNAB budgets with the same credit card
- Transactions have to be split between us
- Both budgets need to be kept in sync manually

<img src="/ynab-journey-orig.svg"/>

<style>
  img {
    @apply absolute bottom-0 right-0 mr-20 mb-5;
    height: 50vh
  }
</style>

---

## The Solution: YNAB Splitter

- Provide a web application where we can categorize and split transactions
- Adjust both of our budgets to keep them in sync
- Provide an audit log to keep track of the history

<img src="/ynab-journey-splitter.svg"/>

<style>
  img {
    @apply absolute bottom-0 text-center mb-5;
    height: 35vh
  }
</style>

---
layout: fact
---

## YNAB Splitter Demo

---

## YNAB-Splitter with Clean Architecture

- Split application into use cases
  - `List Transactions`
  - `List Categories for User`
  - `Approve Transaction`
  - `Get Audit Log`

- Implement every use case in isolation using "Ports and Adapters"

---

## Ports and Adapters

<object type="image/svg+xml" data="/ports_adapters.drawio.svg"></object>

<style>
  object {
    @apply mt-10;
    height: 31vh;
  }
</style>
---

## Ports and Adapters
<object type="image/svg+xml" data="/ports_adapters_classes.drawio.svg"></object>


---

## Ports and Adapters
<object type="image/svg+xml" data="/ports_adapters_classes-flow.drawio.svg"></object>

---

## Defining a "Use Case"

- Defining the "port" interface

```kotlin {}
interface IApproveTransaction {
    fun executeWith(input: ApproveTransactionInput, presenter: ApproveTransactionPresenter)
}
```

- Defining adapters
```kotlin {all|6}
interface ApproveTransactionPresenter {
    fun present(result: ApproveTransactionResult)
}

interface ReadTransactionsRepository {
    fun getTransaction(id: String): Transaction?
}
```

<arrow v-after x1="600" y1="400" x2="460" y2="330" color="#6895FD" width="3" />
<div v-after class="annotation">Domain Entity</div>

<style>
  .annotation {
    @apply absolute top-0 left-0;
    margin-top: 405px;
    margin-left: 605px;
  }
</style>

---

## Implementing a "Use Case"

- Implementing the "use case"
```kotlin {all|2-6|8|10}
class ApproveTransaction(
        private val readTransactionsRepository: ReadTransactionsRepository,
        private val saveTransactionRepository: SaveTransactionRepository,
        private val categoriesRepository: ReadCategoriesRepository,
        // ...
) : IApproveTransaction {
   override fun executeWith(input: ApproveTransactionInput, presenter: ApproveTransactionPresenter) {
          val transactions = loadTransactions(input);
          val auditLogId = approveTransactions(transactions);
          presenter.present(ApproveTransactionResult(auditLogId))
    }
}
```

<arrow v-click="1" x1="750" y1="180" x2="650" y2="180" color="#6895FD" width="3" />
<div v-click="1" class="annotation first">Injected Dependencies</div>

<arrow v-click="2" x1="100" y1="350" x2="200" y2="260" color="#6895FD" width="3" />
<div v-click="2" class="annotation second">Domain Entities</div>

<arrow v-click="3" x1="350" y1="350" x2="280" y2="300" color="#6895FD" width="3" />
<div v-click="3" class="annotation third">Presenting as side effect, no return!</div>

<style>
  .annotation {
    @apply absolute top-0 left-0;
  }
  .first {
    margin-top: 170px;
    margin-left: 755px;
  }
  .second {
    margin-left: 50px;
    margin-top: 355px;
  }
  .third {
    margin-left: 355px;
    margin-top: 355px;
  }
</style>

---

## Implementing a "Use Case" (II)

- Implementing the adapters
```kotlin {all|3|4}
class YnabTransactionRepository(private val transactionsApi: TransactionsApi) : ReadTransactionsRepository {
    override fun getTransaction(id: String): Transaction? {
          val transactionResponse: transactionsApi.getTransactionById(id)
          return transactionResponse.transaction.toTransaction()
    }
}
```

- Implementing the presenter

```kotlin {all|6}
class RestApproveTransactionPresenter : ApproveTransactionPresenter {

    var presentation: ApproveTransactionResultDocument? = null

    override fun present(result: ApproveTransactionResult) {
        presentation = ApproveTransactionResultDocument(result)
    }
}
```

<arrow v-click="1" x1="750" y1="180" x2="650" y2="170" color="#6895FD" width="3" />
<div v-click="1" class="annotation first">YNAB object</div>

<arrow v-click="2" x1="740" y1="230" x2="580" y2="190" color="#6895FD" width="3" />
<div v-click="2" class="annotation second">Mapping to Domain Entity</div>

<arrow v-click="2" x1="740" y1="400" x2="580" y2="380" color="#6895FD" width="3" />
<div v-click="2" class="annotation third">Mapping to REST object</div>

<style>
  .annotation {
    @apply absolute top-0 left-0;
  }
  .first {
    margin-left: 760px;
    margin-top: 170px;
  }
  .second {
    margin-left: 750px;
    margin-top: 230px;
  }

  .third {
    margin-left: 750px;
    margin-top: 390px;
  }
</style>

---

## Calling a "Use Case"

```kotlin {all|1-3|10-17|19|21}
@RestController
@RequestMapping("/api/v1")
class YnabSplitterController(private val readTransactionRepository: ReadTransactionsRepository, ...) {

    @PostMapping("/transactions/{id}/approveSplit")
    fun approveSplitTransaction(@PathVariable("id") transactionId: String, 
                                @RequestBody(required = true) splitRequest: SplitTransactionRequest
    ): ApproveTransactionResultDocument {

        val presenter = RestApproveTransactionPresenter()

        val input = ApproveTransactionInput(splitRequest.transactionIds, splitRequest.transactionSplit, ...)

        val approveTransaction: IApproveTransaction = ApproveTransaction(
            readTransactionRepository, saveTransactionRepository,
            readCategoriesRepository, // ...
        )

        approveTransaction.executeWith(input, presenter)

        return presenter.presentation
    }
}
```

---
layout: fact
---

## Code Demo

<!-- Show screaming Architecture, Mappers, Adapter, Injection -->

---

# Learnings
- Application becomes highly adaptable and well maintainable
  - Easily change to different database
  - Put in security frameworks without touching the core
- Easy tests with Mocks and "in-memory" implementations
- Lots of mapping to different data structures
  - Kotlin makes this a breeze with `data classes` and `extension functions`

<div v-click>

```kotlin
data class TransactionDocument(
        val id: String,
        val actor: SplitterActorDocument,
        val date: LocalDate,
        val amount: Long,
        val category: CategoryDocument?,
        val memo: String?,
        val payee: String?
)
fun Transaction.toDocument(): TransactionDocument =
    TransactionDocument(id, actor.toDocument(), date.toJavaLocalDate(), amount, category?.toDocument(),
                        memo, payee)
}
```
</div>

---

# Challenges
- Lots of boilerplate for simple use cases like `List Transactions`
- What about "nested" use cases?
  - We could have use cases "calling" other use cases
  - This would make testability much harder
  - Instead: Try calling multiple use cases from the outside (e.g. REST controller)

---

# Questions?

- Check out the Code on Github: https://github.com/Excape/ynab-splitter
- Link to these slides: https://github.com/Excape/clean-architecture-talk