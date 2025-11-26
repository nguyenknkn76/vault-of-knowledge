---
tags:
  - design-pattern
---

```ts
/*
  * Singleton Pattern Implementation
  * Info:
  * Basic structure:
  - Singleton
*/  

class Logger {
  private static instance: Logger;
  private constructor(){} // don't allow using NEW to directly access this classs
  static getInstance(): Logger {
    if (!Logger.instance) Logger.instance = new Logger();
    return Logger.instance;
  }
  log(message: string): void {
    console.log(`[LOG]: ${message}`);
  }
}

// Example Usage
const logger1 = Logger.getInstance();
const logger2 = Logger.getInstance();

logger1.log("Nguyen started the app.");
logger2.log("Khoi started the app.");

console.log("Compare logger1 AND logger2::::",logger1 === logger2);
```