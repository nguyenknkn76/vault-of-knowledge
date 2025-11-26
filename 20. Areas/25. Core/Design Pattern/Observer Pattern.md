---
tags:
  - design-pattern
---

```ts
/*
  ! Observer Pattern Implementation
  * Info: 
  - Behavioral pattern
  * Basic structure:
  - Observer interface
  - Concrete observers
  - Subject interface
  - Concrete subject
*/

/*
  Example:
  * Observer Pattern in a Stock Market Application
  - Stock: Subject that notifies observers about price changes.
  - Trader, Dashboard: Observer that recevies updates from Stock(Subject).
*/

// Observer interface
interface StockObserver {
  update(price: number): void;
}

// Subject interface
interface StockSubject {
  addObserver(observer: StockObserver): void;
  removeObserver(observer: StockObserver): void;
  notifyObservers(): void;
  setPrice(price: number): void;
}

// Subject concrete
class Stock implements StockSubject{
  private observers: StockObserver[] = [];
  private price: number = 100;

  addObserver(observer: StockObserver): void{
    this.observers.push(observer);
  }
  removeObserver(observer: StockObserver): void {
    this.observers = this.observers.filter(obs => obs !== observer);
  }
  setPrice(newPrice: number): void {
    this.price = newPrice;
    this.notifyObservers();
  }
  notifyObservers(): void {
    for (const observer of this.observers){
      observer.update(this.price);
    }
  }
}

// Concrete observer
class Trader implements StockObserver {
  constructor(private name: string){}
  update(price: number): void {
      console.log(`${this.name} notified of price change: ${price}`);
  }
}

class Dashboard implements StockObserver {
  update(price: number): void {
      console.log(`Dashboard updated with new price: ${price}`);
  }
}

// Example usage
const appleStock = new Stock();
const traderA = new Trader("Trader A");
const traderB = new Trader("Trader B");
const dashboard = new Dashboard();

appleStock.addObserver(traderA);
appleStock.addObserver(traderB);
appleStock.addObserver(dashboard);

appleStock.setPrice(120);
appleStock.setPrice(150);



```