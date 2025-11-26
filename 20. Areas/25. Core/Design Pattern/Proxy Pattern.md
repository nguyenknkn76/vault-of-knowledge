---
tags:
  - design-pattern
---

```ts
/*
  ! Proxy Pattern Implementation
  * Info: 
  * Basic structure: 
  - Proxy class
  - Subject interface
  - RealSubject class
  - Client
*/

/*
  Example:
  * File Manager Proxy
*/

// Interface 
interface IFileManager {
  deleteFile(fileName: string): void;
}

// Real Subject Class
class RealFileManager implements IFileManager {
  deleteFile(fileName: string): void {
      console.log(`File ${fileName} deleted successfully.`);
  }
}

// Proxy
class FileManagerProxy implements IFileManager {
  private realFileManager = new RealFileManager();
  constructor(private userRole: "admin" | "guest"){}
  deleteFile(fileName: string): void {
      if (this.userRole !== "admin") console.log("Access denied. Only admins can delete files.");
      else this.realFileManager.deleteFile(fileName);
  }
}

// Exmaple Usage
const adminProxy = new FileManagerProxy("admin");
adminProxy.deleteFile("example.txt");

const guestProxy = new FileManagerProxy("guest");
guestProxy.deleteFile("example.txt");
```