---
tags:
  - design-pattern
---
```ts
/*
  ! Bridge Pattern Implementation
  * Info: Structural
  * Basic structure
  - Abstraction interface
  - Refined Abstraction
  - Implementor interface
  - Concrete Implementor 
*/

/*
  Example: 
  * Notification System
*/  

// Implemetation interface
interface MessageSender {
  send(message: string): void;
}
// Concreate Implementor
class EmailSender implements MessageSender{
  send(message: string): void {
      console.log(`Send Email:::::`, message);
  }
}
class SlackSender implements MessageSender{
  send(message: string): void {
      console.log(`Send Slack:::::`, message);
  }
}

// Abstraction
class SystemNotification {
  constructor(protected sender: MessageSender){}
  sendMessage(content: string): void{
    this.sender.send(content);
  }
}
// Refined Abstraction
class AlertNotification extends SystemNotification{
  sendAlert(): void{
    this.sendMessage("System alert");
  }
}
class AuthNotification extends SystemNotification{
  sendAuthCode(): void {
    this.sendMessage("Auth Code: 123456");
  }
}

// Example Usage
const emailAlert = new AlertNotification(new EmailSender());
emailAlert.sendAlert();
const slackAuth = new AuthNotification(new SlackSender());
slackAuth.sendAuthCode();
```