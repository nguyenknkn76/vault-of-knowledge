---
tags:
  - design-pattern
---
```ts
/*
  ! Strategy Pattern Implementation
  * Info:
  - Behavioral pattern
  * Basic struture 
  - Strategy interface
  - Concrete strategies
  - Context class
*/

/*
  Example: 
  * Strategy Pattern in a Payment Processing System
  - PaymentStrategy: Strategy interface for different payment methods.
  - CreditCardPayment, CashPayment, EWalletPayment: Concrete strategies implementing the PaymentStrategy interface.
  - PaymentProcessor: Context class that uses a strategy to process payments.
  - Strategy Map: An alternative way to manage strategies using a map.
*/
// Strategy interface
interface PaymentStrategy {
  pay(amount: number): void;
};

// Concrete strategies
class CreditCardPayment implements PaymentStrategy {
  pay(amount: number) : void {
    console.log(`Paid ${amount} using Credit Card.`);
  }
}

class CashPayment implements PaymentStrategy {
  pay(amount: number): void {
      console.log(`Paid ${amount} using Cash.`);
  }
}

class EWalletPayment implements PaymentStrategy {
  pay(amount: number): void {
      console.log(`Paid ${amount} using E-Wallet.`);
  }
}

// Context class
class PaymentProcessor {
  private strategy: PaymentStrategy;
  
  constructor(strategy: PaymentStrategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy: PaymentStrategy): void {
    this.strategy = strategy;
  }

  processPayment(amount: number): void {
    this.strategy.pay(amount);
  }
}

// Example usage
const processor = new PaymentProcessor(new CreditCardPayment());
processor.processPayment(100);
processor.setStrategy(new CashPayment());
processor.processPayment(50);


/*
  * Using Strategy Map

type PaymentMethod = 'credit' | 'cash' | 'ewallet';
const strategyMap: Record<PaymentMethod, PaymentStrategy> = {
  credit: new CreditCardPayment(),
  cash: new CashPayment(),
  ewallet: new EWalletPayment()
}

const processPayment = (method: PaymentMethod, amount: number): void => {
  const strategy = strategyMap[method];
  if (!strategy) throw new Error(`Payment method ${method} not supported.`);
  strategy.pay(amount);
}

// Example usage
console.log('Using Strategy Map:::Results')
processPayment('credit', 100);
processPayment('cash', 50);
processPayment('ewallet', 75);

*/
```