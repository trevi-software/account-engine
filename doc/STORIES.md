
# Ledger
The ledger is the "engine" that records and tracks the financial position of any orgainization or entity. It
is the ultimate goal of this project to provide the "accounting engine" that another larger application can
use drive its platform. It is meant to be used by ERPs, Fintech applications, financial tracking and
management applications.

#### Card
As an accountant in charge of setting up my company's accounts I would like to create and maintain a ledger
so I and my bookkeepers can create and maintain the company's financial records.

#### Conversation
- Non-goal: not interested in providing a GUI
- Non-goal: not interestedin in providing an API
- However, it should be possible for another app to slap a GUI or an API on this library and provide a more-or-less
complete accounting program out of the box
- Implement basic accounting sanity checks
  - double-entry accounting
  - transactions: ensure Dr and Cr match
  - transactions: ensure accounting equation is not violated
- How do we handle adjusting entries (should they even be included in this paradigm) ?
- How do we handle the "Accounting Cycle" ?

#### Acceptance Criteria
Given the ledger should support standard accounting practices, such as GAAP and IFRS, when a ledger is created, then
it will support the following functionality:
- create an arbitrary number of accounts
- edit account information
- de-activation of an account
- prevent deletion of an account that has a transaction recorded in it
- list the accounts in the ledger
- lock an account to prevent modification or creation of any records prior to and including a given date
- create a Trial Balance of the ledger at any current or past date
- create an Income Statement from the ledger at any current or past date
- create a Balance Sheet from the ledger at any current or past date
- create an arbitrary number of journals to record financial transactions
- edit journal information
- de-activation of a journal
- prevent deletion of a journal which has a transaction recorded in it
- lock a journal to prevent modification or creation of any transactions prior to and including a given date
- record a financial transaction in the journal
- post a transaction from a journal to the ledger
- support storage backends: MemoryStore and PostgreSQL

## Epic - Basic Ledger

### Create a ledger
#### Card
As an accountant I want to define some basic aspects of the ledger so that it does not need to be duplicated in
other places.
#### Conversation
- Use interim and leaf accounts to model the ledger hierarchy. Leaf accounts record transactions. Interim accounts
aggregate the Dr and Cr columns of the accounts below them.
- Accounts should be arranged by type: Assets, Liabilities, Capital, Revenue, Expense
#### Acceptance Criteria
Given the caller supplies all required information, when a ledger is created, it will:
- have a name
- have a default accounting period end-date defined by the month and day of the last day of the period
- have zero or more default interim periods defined
- have an one accounting period, which will be the current period. It's start and end dates will
be defined relative to the default accounting period end-date.
- have a root account with the name of the ledger, who's only purpose is to function as parent to all first level accounts
- have interim accounts defined as such:
Root
  Assets
  Liabilities
  Capital
  Revenue
  Expenses
- have no leaf accounts defined
- have no journals defined

### Create an account
#### Card
As an accountant I want to create an account so that I can record and track financial transactions related to
the company and outside entities that have a financial relationship with it.
#### Conversation
- Accounts recording information about the company are called **Ledger Accounts**. Accounts that refer to
outside entities are called **External Accounts**.
- Account numbering should be left up to the caller/user.
- Account records have two types of transactions: Debits (Dr) and Credits (Cr)
- Account balances can be derived from transactions, but there are instances (i.e- end of period) when
they need to be recorded. Usually required by law (i.e financial reporting, bank account balance statements).
- records should be sorted in chronological order
#### Acceptance Criteria
Given the caller supplies all required information, when an account is created, then it will:
- have a name
- have an account number
- identify whether it is a ledger or an external account
- have no transactions recorded

### Create a journal
#### Card
As an accountant I want to create a journal so a bookkeeper can record the company's financial transactions
as they happen.
#### Conversation
- Should contain an unique identifier for posting purposes
- transactions are sorted in chronological order
#### Acceptance Criteria
Given the caller supplies all required information, when a journal is created, then it will:
- have a name
- have a code

### Create a journal transaction (xact)
#### Card
As a bookkeeper I want to record financial transactions as they happen in a journal so they do not
get forgotten.
#### Conversation
- Need to keep track of draft and posted transactions
- Once a transaction is posted it cannot be modified or deleted
#### Acceptance Criteria
Given the caller supplies all required information, when a transaction is created, then it will:
- have a unique identifier
- be in an Open state
- have a date that is less than or equal to today but greater than the date of the last locked period
- have a Dr account and a Cr account that are **NOT** the same account
- have a transaction amount that is supplied by the caller or zero, otherwise
- have a description of the transaction
- have an empty Posting Reference field
- be freely modifyable (or even deleted)

When a transaction is posted:
- the Posting Reference field will contain the Dr and Cr account number separated by a comman
- be in a Posted state
- the library must refuse to modify or delete the transaction

### Create a ledger record (journal entry)
### Card
As an accountant I must be able to post transactions recorded in a journal to the ledger
accounts affected by the transaction so that they are reflected accurately in the ledger.
### Conversation
- ~~should the records be derived from the journals at run-time or should they be copied into
a separate record.~~ For immutability purposes database records should be created.
- once created the record should not be deleted or modified
### Acceptance Criteria
Given a complete journal transaction record, when a journal entry is created, then it will:
- have a Journal Reference containing the unique identfier of the journal transaction
- have a date identical to the date in the journal transaction
- have a non-deactivated account
- if the account was debited in the journal transaction the Dr field of the entry will
have an amount identical to the one in the journal transaction and the Cr field will be empty and
the description will contain the name of the account that was credited
- if the account was credited in the journal transaction the Cr field of the entry will
have an amount identical to the one in the journal transaction and the Dr field will be empty and
the description will contain the name of the account that was debited
- refuse to modify or delete it

### Post one or more journal transactions to the ledger
### Card
As an accountant I want to post one or more journal transactions to the ledger so that they can be
accurately reflected in the ledger accounts.
### Conversation
see discussion above
### Acceptance Criteria
Given a full and accurate journal transaction, when the caller posts it to the ledger, then it will perform the following steps:
- Each journal transaction will create two journal entries in the ledger
1. Initiate a post to the Dr account in the transaction and
   - Copy the transaction date from the journal to the journal entry
   - Copy the journal unique identifier to the "Journal Reference" field
   - Copy the amount of the tranaction to the Dr field
2. Initiate a post to the Cr account in the transaction and
   - Copy the transaction date from the journal to the journal entry
   - Copy the journal unique identifier to the "Journal Reference" field
   - Copy the amount of the tranaction to the Cr field
3. Copy the account number from the Dr and Cr accounts, separated by a comma
4. Set the state of the journal transaction to posted

