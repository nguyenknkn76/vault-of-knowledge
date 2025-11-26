---
tags:
  - design-pattern
---

```ts
/*
  ! Adapter Pattern Implementation
  * Info: Structural
  * Basic structure
  - Target
  - Adapter
  - Adaptee interface
  - Concrete Adaptee
*/

/*
  Example:
  * Adapter to Convert Temperature Units
*/  

// Adaptee interface
interface IFahrenheitSensor {
  getTemperature(): number;
}

// Adaptee (concrete)
class FahrenheitSensor {
  getTemperature(): number{
    return 86;
  }
}

// Target interface
interface CelsiusSensor {
  getCelsius(): number;
}

// Adapter
class FahrenheitAdapter implements CelsiusSensor{
  private sensor: FahrenheitSensor;
  constructor(sensor: FahrenheitSensor){
    this.sensor = sensor;
  }
  getCelsius(): number {
    const f = this.sensor.getTemperature();
    const c = ((f-32)*5)/9;
    return c;
  }
}

// Example Usage
const fahrenheitSensor = new FahrenheitSensor();
const adapter = new FahrenheitAdapter(fahrenheitSensor);
console.log(`Current temperature:::::`, adapter.getCelsius());
```