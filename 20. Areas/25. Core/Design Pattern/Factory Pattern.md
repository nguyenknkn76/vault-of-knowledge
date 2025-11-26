---
tags:
  - design-pattern
---

## 1. Factory Abstract Pattern

```ts
/*
  * Abstract Factory Pattern Implementation
  * Info: Creatational
  * Basic structure:
  - Client
  - Abtract Factory interface
  - Concrete Factory class
  - Abtract Product interface
  - Concrete Product class
*/

// Product Interface
interface Button {
  render(): void;
}

interface Checkbox {
  render(): void;
}

// Concrete Product class
class IOSButton implements Button {
  render(): void {
      console.log("IOS Button");
  }
}
class AndroidButton implements Button {
  render(): void {
      console.log("Android Button");
  }
}
class IOSCheckBox implements Checkbox {
  render(): void {
      console.log("IOS Checkbox");
  }
}
class AndroidCheckBox implements Checkbox {
  render(): void {
      console.log("Android Checkbox");
  }
}

// Abtract Factory interface
interface UIFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
}

// Concrete Factory
class IOSFactory implements UIFactory{
  createButton(): Button {
      return new IOSButton();
  }
  createCheckbox(): Checkbox {
      return new IOSCheckBox();
  }
}
class AndroidFactory implements UIFactory{
  createButton(): Button {
      return new AndroidButton();
  }
  createCheckbox(): Checkbox {
      return new AndroidCheckBox();
  }
}

// Example Usage
const buildUI = (factory: UIFactory) => {
  const button = factory.createButton();
  const checkbox = factory.createCheckbox();
  button.render();
  checkbox.render();
}

const aFactory = new AndroidFactory();
buildUI(aFactory);
const iFactory = new IOSFactory();
buildUI (iFactory);
```

## 2. Factory Method Pattern

```ts
/*
  ! Factory Method Pattern Implementation
  * Info:
  * Basic structure:
  - Product interface
  - Concrete product
  - Concrete creator
  - Creator abtract class
  - Client
*/

// Note: prefix "M" → don't care about it → just use it to distinguish Factory Simple Pattern and Factory Method Pattern
// Product interface
interface MDelivery {
  ship(): void;
}

// Concreate 
class MLocalDelivery implements MDelivery {
  ship(): void {
      console.log("Shipping Locally");
  }
}

class MRemoteDelivery implements MDelivery {
  ship(): void {
    console.log("Shipping Remotely");
  }
}

class MInternationalDelivery implements MDelivery {
  ship(): void {
      console.log("Shipping Interationaly");
  }
}

// Abtract Creator
abstract class MDeliveryCreator {
  abstract createDelivery(): MDelivery;
  deliver(): void {
    const delivery = this.createDelivery();
    delivery.ship();
  }
} 

// Concrete Creator
class MLocalDeliveryCreator extends MDeliveryCreator{
  createDelivery(): MDelivery {
    return new MLocalDelivery();
  }
}
class MRemoteDeliveryCreator extends MDeliveryCreator{
  createDelivery(): MDelivery {
    return new MRemoteDelivery();
  }
}
class MInternationalDeliveryCreator extends MDeliveryCreator{
  createDelivery(): MDelivery {
      return new MInternationalDelivery();
  }
}

// Example Usage
const localCreator = new MLocalDeliveryCreator();
localCreator.deliver();

const remoteCreator = new MRemoteDeliveryCreator();
remoteCreator.deliver();

const internationalCreator = new MInternationalDeliveryCreator();
internationalCreator.deliver();
```

## 3. Factory Simple Pattern

```ts
/*
  ! Factory Simple Pattern Implementation
  * Info:
  * Basic structure:
  - Factory class
  - Product interface
  - Concrete product classes
*/

/*
  Example:
  *
*/

// Product interface
interface Delivery {
  ship(): void;
}

// Concrete product classes
class LocalDelivery implements Delivery {
  ship(): void {
      console.log("Shipping locally");
  }
}

class RemoteDelivery implements Delivery {
  ship(): void {
    console.log("Shipping Remotely");
  }
}

class InternationalDelivery implements Delivery {
  ship(): void {
      console.log("Shipping Interationaly");
  }
}

// Factory function
const DeliveryFactory = (type: string): Delivery => {
  switch (type){
    case "local":
      return new LocalDelivery();
    case "remote": 
      return new RemoteDelivery();
    case "internationtal":
      return new InternationalDelivery();
    default:
      throw new Error("Unidentify delivery methods");
  }
}

const myDelivery = DeliveryFactory("remote");
myDelivery.ship();
```