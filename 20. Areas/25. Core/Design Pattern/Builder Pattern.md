---
tags:
  - design-pattern
---

```ts
/*
  ! Builder Pattern Implementation
  * Info: Creational
  * Basic structure
  - Director class
  - Builder interface (abtract)
  - Concrete builder
  - Product
*/

/*
  Example:
  * Create Advanced User Profile
*/
// Product
class UserProfile {
  constructor(
    public name: string,
    public email: string,
    public age?: number,
    public address?: string,
    public phone?: string
  ) {}
}

// Builder Interface
interface IUserBuilder {
  setName(name: string): IUserBuilder;
  setEmail(email: string): IUserBuilder;
  setAge(age: number): IUserBuilder;
  setAddress(address: string): IUserBuilder;
  setPhone(phone: string): IUserBuilder;
  build(): UserProfile;
}

// User Builder (concrete)
class UserBuilder implements IUserBuilder {
  private name!: string;
  private email!: string;
  private age?: number;
  private address?: string;
  private phone?: string;

  setName(name: string): IUserBuilder {
    this.name = name;
    return this;
  }
  setEmail(email: string): IUserBuilder {
    this.email = email;
    return this;
  }
  setAge(age: number): IUserBuilder {
    this.age = age;
    return this;
  }
  setAddress(address: string): IUserBuilder {
    this.address = address;
    return this;
  }
  setPhone(phone: string): IUserBuilder {
    this.phone = phone;
    return this;
  }
  build(): UserProfile {
    return new UserProfile(this.name, this.email, this.age, this.address, this.phone);
  }
}

// Director
class UserDirector {
  constructor(private builder: IUserBuilder){}
  createMinimal(name: string, email: string): UserProfile{
    return this.builder
      .setName(name)
      .setEmail(email)
      .build();
  }
  createFull(
    name: string, 
    email: string, 
    age: number, 
    address: string, 
    phone: string
  ): UserProfile{
    return this.builder
      .setName(name)
      .setEmail(email)
      .setAge(age)
      .setAddress(address)
      .setPhone(phone)
      .build();
  }
}

// Example Usage
const builder = new UserBuilder();
const director = new UserDirector(builder);

const basicUser = director.createMinimal("Nguyen","nguyen@example.com");
const fullUser = director.createFull(
  "Nguyen",
  "nguyen@example.com",
  23,
  "Ha Noi",
  "0123456789"
);
console.log(`Basic User:::::`, basicUser);
console.log(`Full User:::::`,fullUser);
```




