# budget
My budgeting system.

## Introduction

Around 2016, I decided to start better keeping track of my expenses. I started with the [Bluecoins app](https://www.bluecoinsapp.com/), manually adding my daily expenses but not doing much with it - after all, my income back then was just a grad school scholarship. I didn't have that many financial responsibilities either, so I lacked critical understanding of what makes budgeting important.

In 2018, shortly after starting at my first job, I took a critical step: I started a three-month trial of [You Need a Budget](https://www.ynab.com/). I loved the system! The app was easy to use and, even without bank integration in Brazil, I got into the rhythm of tracking _everything_ there. The envelope-based system resonated with me and it was honestly fun.

That is, until they decided to jack up the prices. With no regional pricing in place, it made no financial sense for me to continue paying for it (which is ironic, considering it essentially allowed me to understand that by the very virtue of it working). By then, I was also starting to see the parts that weren't so good for my use case - for example, it's not good at multicurrency scenarios, an important feature for me when it comes to international travel or importing stuff.

By 2020 I started using my custom Excel spreadsheet - built from the ground up with purely native functions and based on the YNAB system, but adapted to my needs and desires. In three years it has grown to be a multi-currency ledger with budgeting and scheduling. It is also cumbersome due to being, well, an evergrowing MVP (trust me, you don't want to see what needs to be done to add a new currency).

My goal here is to accurately synthesize my organically-created budgeting system into human-readable documentation, to assist me in better understanding what I want and to hopefully guide a solid next-gen implementation.

## Foundational Concepts

### Mixing the Envelope and Zero-Based Budgeting Systems

The philosophy of this system is: all of your money is put into envelopes. These envelopes are labeled according to the categories they represent - "groceries", "school", "emergency fund", "travel"... No money lives outside of an envelope - as soon as you earn anything, into envelopes it goes.

You can move money between envelopes - as YNAB says, "roll with the punches". The thing is that this needs to be a _conscious, deliberate decision_ - you are taking money away from one thing to handle another. Speaking of YNAB... I think their Four Rules are a great foundation and to this date are the basis of my implementation. [Go check them out!](https://www.ynab.com/the-four-rules)

### Installments

One of the most common pieces of financial advice for beginners is: always pay in one go. The reason behind this is sound: installments can make you believe you have spent just a fraction of that big number, when in fact you are _indebted_ to a financial entity - a bank.

Banks are more than happy to let you believe that, since that's how they make money. As the saying goes, the ~house~ bank always wins, so you better be on top of this debt by paying installments by their due date. This is harder than it sounds when you are used to track your budget by looking directly at your bank statements. Envelopes, however, make that extremely easy with two rules:

1. **You always take the full amount of an expense from the envelopes.** It doesn't matter if you are paying in one hundred installments, from the perspective of your budget you are always paying the full amount at once. This leads us to the second rule...
2. **You only spend what you already have.** If you need to take the full value of an expense from the envelopes, you need to have the amount in them in the first place. This is a surefire way of avoiding counting your chickens before they hatch, i.e., spending money you don't yet have just because the credit card statement will come _someday_ in the future.

## Abstractions

Before we get into the elements that compose this budgeting system, let's talk about abstraction levels - or, as I call them, _layers_. A layer is a modeling of your budget; it allows you to freely move money from one place to another and convert units, just like you can do in practice.

One layer doesn't know how money is organized in the others, nor does it know about any of its operations - they only share how much money they contain and in which currencies such money exists. This allows us to freely organize money in different ways, which is the main reason for this abstraction to exist in the first place.

The system is composed by two layers: **physical** and **logical**. These two together aim to give you the tools to be able to answer the _hows_ of life: how to buy a house, how to pay debt off, how to save for the future, and anything else you can dream of.

### Physical layer, or: _where?_

The foundation of a budgeting system is how it models the real world. That means representing your actual assets - bank accounts, physical wallets, and anything else that societies and financial systems leverage.

In other words, in this layer we model _where_ the money is.

Knowing where the money is is extremely important, and having a single pane of glass to cover multiple assets provides convenience and facilitates the application of best financial practices.

### Logical layer, or: _why?_

Just having the information about your assets isn't smart enough, though: that's just wealth and expense tracking. To _truly_ manage your financial life, you need money to have _purpose_ - and it is exactly what the logical layer does.

Here, we model the _whys_ of money: why you have it, why it will be spent, why it is being spent.

## Core Elements

Now that we understand the layers, we can talk about the things that make them up.

### Balance

![image](https://github.com/altifuse/budget/assets/29523470/8f63efce-8783-48e5-92ca-b1938bf227a9)

Money can't exist in a vacuum, it needs to always be contained _somewhere_. That's a _Balance_. In this system, we have:
* _Physical Balances_, which are... 
  * **Assets**: as previously explained, a bank account, a wallet, position in a stock, etc; it's a place where money lives.
  * **Liabilities**: any debts. Can be thought of as assets but that hold negative amounts by default. Credit cards are liabilities!
* _Logical Balances_, aka **Categories**: since the logical layer aims to answer the _whys_ of money, a Balance in this abstraction is what the famous, eponymous budgeting method calls _envelope_.

There are special types of Balances as well, and these apply to both layers:
* **Sources**: bottomless, valueless Balances from which money can only be taken.
* **Sinks**: bottomless, valueless Balances to which money can only be added.

Sources and Sinks are used to represent third parties. For example, your employer is represented by a Source, while a grocery store is represented by a Sink.

In the logical layer, there is one special instance of Category: **To Budget**. This Category contains, by default, the amount from the physical layer that exceeds the amount in the totality of user-defined Categories.

### Currency

A special attribute that every Balance contains is Currency. A Balance instance can only have a single Currency: if you want to, for example, map a multicurrency bank account (like a Wise Borderless account), you need one Balance for each currency. This limitation makes it easy to understand and manage the next element.

### Transaction

![image](https://github.com/altifuse/budget/assets/29523470/a8f6d153-bf24-4721-ab8b-b24eaf55007b)

If Balances are the places, Transactions are the act of moving money between them. At its core, a Transaction can only represent a one-to-one operation - that means one-to-many, many-to-one, and many-to-many operations need to be represented as multiple Transactions.

We have the following Transaction types:
* **Income**: Transaction from a Source to an Asset. The Category is always a hard-coded one called **Unallocated**.
* **Expense**: Transaction from an Asset to a Sink. There's always a source Category, and the destination logical Sink matches the physical one.
* **Transfer**: simple Transaction between physical Balances and covering amounts of the same Currency.
* **Allocation**: simple Transaction between Categories and covering amounts of the same Currency.
* **Exchange**: Transaction from one Asset to another Asset with a set Category. This always moves money between Currencies. (yes, a Category, being a Balance, can only have one Currency - but multiple Category instances can be grouped up as one.)

### Budget

![image](https://github.com/altifuse/budget/assets/29523470/89708c2c-4613-4fd0-a042-dbafc82a1876)

There are good reasons to keep parts of your financial life separate from one another: investment assets should not be easily mixed up with day-to-day expenses, and saving for that extended vacation abroad might require you to create lots of different Categories.

It is therefore possible to keep multiple Budgets in the system: each Budget contains its own Balances and Transactions and, therefore, isolated total amounts.

## Budgeting: Contributions

### Scheduled Contribution

Every fixed expense needs to be planned for and budgeted ahead of time. This is implemented by having the system evaluate every period for new contributions to the category and requesting new contributions until the value is matched.

#### Examples
* Fixed recurring expenses, such as streaming
* Variable but predictable recurring expenses, such as grocery shopping and utilities
* Savings targets, such as investment plans

#### Attributes
* Periodicity (yearly, monthly, weekly)
* Due date within the period
* Category
* Value

>**_💡 Food For Thought_**
>
> _In scheduled contributions (and its subtypes), imagine this scenario: you reach the target contribution but, within the same evaluation period, you move money out of the Category (not by spending it, but by moving to another Category). Does that count against the goal or not?_

### Target Balance

Categories that should always have a set amount budgeted. If balance is used for expenses, transferred out, or the target raises, new contributions will be requested until the value requirement is fulfilled again.

#### Examples
* Emergency funds
 
#### Attributes
* Category
* Value

### Target Balance by Date

An offshoot of the scheduled contribution, but with contribution split into installments. In every budget period, an installment will be calculated by subtracting the currently allocated value from the target value and dividing that by the number of periods until the target date.

#### Examples
* Planned, large one-time expenses, such as vacations and big purchases
* Fixed/predictable recurring expenses - but specifically those with longer periods between recurrences, like yearly subscriptions

#### Attributes
* Due date
* Category
* Value

