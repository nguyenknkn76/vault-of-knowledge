---
tags:
  - design-pattern
---

```ts
/*
  ! Prototype Pattern Implementation
  * Info: Creational 
  * Basic struture: 
  - Prototype interface
  - Concrete prototype
  * 2 Technique: `deep copy` OR `shallow copy`
*/

// Prototype interface
interface OrderPrototype {
  clone(): OrderPrototype;
  display(): void;
}

// Concrete prototype
class Order implements OrderPrototype {
  constructor(
    public region: string,
    public shippingMethod: string,
    public paymentType: string, 
    public items: string[],
    public customerName?: string
  ){}
  clone(): OrderPrototype {
      return new Order(
        this.region,
        this.shippingMethod,
        this.paymentType,
        [...this.items] // clone array
      );
  }
  display(): void {
    console.log(
      `customerName:::${this.customerName ?? "__empty_name__"}
      region:::${this.region}
      shippingMethod:::${this.shippingMethod}
      paymentType:::${this.paymentType}
      items::::${this.items.join(", ")}`
    )
  }
}

// Example Usage
const hanoiOrderTemplate = new Order(
  "Ha Noi",
  "Fast delivery",
  "COD",
  ["T-shirt", "Sneaker", "Basketball"]
);

const nguyenOrder = hanoiOrderTemplate.clone();
(nguyenOrder as Order).customerName = "Nguyen";
nguyenOrder.display();

const khoiOrder = hanoiOrderTemplate.clone();
(khoiOrder as Order).customerName = "Khoi";
khoiOrder.display();

const anonymousOrder = hanoiOrderTemplate.clone();
anonymousOrder.display();
```