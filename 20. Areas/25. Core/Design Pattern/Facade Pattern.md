---
tags:
  - design-pattern
---

```ts
/*
  * Facade Pattern Implementation
  * Info: Structural pattern
  * Basic structure: 
  - Facade class
  - Subsystem classes
*/

/*
  Example:
  * Facade Pattern in a Home Theater System
  - HomeTheaterFacade: simplifies the interaction with various subsystem classes like Projector, SoundSystem, and Lights.  
  - Subsystem classes: Projector, SoundSystem, and Lights that perform specialized tasks.
*/

// Subsystem classes
class Projector {
  on() {console.log("Projector ON");}
  off(){console.log("Projector OFF");}
}

class SoundSystem {
  on() {console.log("Sound System ON");}
  off() {console.log("Sound System OFF");}
}

class Lights {
  dim() {console.log("Lights DIMMED");}
  on() {console.log("Lights ON");}
}

// Facade class
class HomeTheaterFacade {
  private projector = new Projector();
  private sound = new SoundSystem();
  private lights = new Lights();

  startMovie(): void {
    console.log("Starting Movie...");
    this.lights.dim();
    this.projector.on();
    this.sound.on();
  }

  endMovie(): void {
    console.log("Ending Movie...");
    this.sound.off();
    this.projector.off();
    this.lights.on();
  }
}

// Example usage
const theater = new HomeTheaterFacade();
theater.startMovie();

setTimeout(()=> {
  theater.endMovie();
}, 3000);
```