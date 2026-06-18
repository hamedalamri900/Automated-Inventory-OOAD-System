# Automated-Inventory-OOAD-System
A complete Object-Oriented Analysis and Design (OOAD) architecture for an Automated Inventory & PO System, including UML diagrams.

## 📌 System Overview
The system is designed strictly using Object-Oriented Analysis and Design principles to manage warehouse inventory. By representing system entities as interacting objects (e.g., `User`, `Product`, `PurchaseOrder`), the system encapsulates business logic, eliminates human errors, and prevents sudden product shortages. 

When a `Product` object's stock reaches its predefined "Reorder Point", the system auto-instantiates a new `PurchaseOrder` object, routes it to a `FinanceManager` for approval, and transmits it to an external `SupplierAPI`.

## 🏗️ Object-Oriented Principles Applied
* **Encapsulation:** All sensitive data (product quantities, financial limits) are hidden within their respective classes and can only be accessed/modified through secure public methods.
* **Inheritance:** A base abstract class `User` is created. Specific actors like `InventoryManager` and `FinanceManager` inherit from this base class, adding their specific roles.
* **Polymorphism:** The system utilizes interfaces for the `SupplierAPI`, allowing communication with multiple external suppliers using a unified method signature.

---

## 📊 System Modeling Diagrams

### 1️⃣ UML Use Case Diagram
```mermaid
flowchart LR
    classDef actor fill:#f9f9f9,stroke:#333,stroke-width:1.5px,color:#000;
    classDef usecase fill:#ffffff,stroke:#0288d1,stroke-width:1.5px,color:#000;

    IM(("Inventory\nManager")):::actor
    FM(("Finance\nManager")):::actor
    SA(("System\nAdmin")):::actor
    Sys(("System\nListener")):::actor
    API(("Supplier\nAPI")):::actor

    subgraph System ["Automated Inventory & PO System"]
        direction TB
        UC01(["UC-01: Set Reorder Point & Stock Limits"]):::usecase
        UC02(["UC-02: Auto-Instantiate Draft PO"]):::usecase
        UC03(["UC-03: Review and Transmit PO"]):::usecase
        UC04(["UC-04: Receive PO Shipment"]):::usecase
        UC05(["UC-05: Instantiate New Product"]):::usecase
        UC06(["UC-06: Reject Purchase Order"]):::usecase
        UC07(["UC-07: Instantiate User Profiles"]):::usecase
        UC08(["UC-08: Audit System Logs"]):::usecase
        UC09(["UC-09: Acknowledge PO Receipt"]):::usecase
        UC10(["UC-10: Generate Inventory Reports"]):::usecase
    end

    IM --- UC01
    IM --- UC04
    IM --- UC05
    IM --- UC10

    FM --- UC03
    FM --- UC06
    FM --- UC10

    Sys --- UC02
    UC02 -.->|include| UC03

    SA --- UC07
    SA --- UC08

    UC03 -.->|transmit| API
    API --- UC09
```

### 2️⃣ UML Class Diagram (Core OOP Structure)
```mermaid
classDiagram
    class User {
        <<abstract>>
        -String userID
        -String name
        -String email
        -String passwordHash
        +login(credentials: String) Boolean
        +logout() void
    }

    class InventoryManager {
        +instantiateProduct(sku: String, name: String) Product
        +setReorderThresholds(product: Product, min: int, max: int) void
        +updateStock(product: Product, qty: int) void
        +receiveShipment(po: PurchaseOrder) void
    }

    class FinanceManager {
        -Double approvalLimit
        +getPendingPOs() List~PurchaseOrder~
        +approvePO(po: PurchaseOrder) void
        +rejectPO(po: PurchaseOrder, reason: String) void
    }

    class SystemAdmin {
        +instantiateUserProfiles() User
        +assignRole(user: User, role: String) void
        +auditSystemLogs() List~AuditLog~
    }

    User <|-- InventoryManager : Inherits
    User <|-- FinanceManager : Inherits
    User <|-- SystemAdmin : Inherits

    class Product {
        -String SKU
        -String name
        -Double price
        -Integer currentStock
        -Integer reorderPoint
        -Integer maxStock
        +updateStock(qty: Integer) void
        +isLowStock() Boolean
        +calculateRequiredQuantity() Integer
        +setReorderPoint(min: Integer) void
    }

    class PurchaseOrder {
        -String poID
        -String state
        -Integer requiredQuantity
        -String rejectionReason
        +approve() void
        +rejectPO(reason: String) void
        +receiveShipment() void
        +changeState(newState: String) void
    }

    class AuditLog {
        -String logID
        -String userID
        -DateTime timestamp
        -String actionDetails
        +recordMutation() void
    }

    class SupplierAPI {
        <<interface>>
        +transmitPO(po: PurchaseOrder) Boolean
        +parseResponse(response: Payload) String
    }

    Product "1" <-- "0..*" PurchaseOrder : Contains
    PurchaseOrder ..> SupplierAPI : Transmitted via
    User "1" --> "0..*" AuditLog : Generates Actions
    InventoryManager ..> Product : Manages
    FinanceManager ..> PurchaseOrder : Evaluates
    SystemAdmin ..> AuditLog : Reviews
```

### 3️⃣ UML Sequence Diagram
```mermaid
sequenceDiagram
    autonumber
    
    actor Sys as System Listener
    participant P as :Product
    participant PO as :PurchaseOrder
    actor FM as Finance Manager
    participant API as <<interface>> SupplierAPI

    Note over Sys, P: 1. Sales Transaction
    Sys->>P: updateStock(qty)
    activate P
    P->>P: isLowStock()
    
    alt Stock <= reorderPoint
        P->>P: calcRequiredQty()
        P-->>Sys: Return (True, qty)
        
        Note over Sys, PO: 2. Auto-Instantiate PO
        Sys->>PO: <<create>> new PO(Product, qty)
        activate PO
        PO-->>Sys: PO Created (Draft)
        Sys->>FM: notifyPending(PO)
    else Stock > reorderPoint
        P-->>Sys: Return (False)
    end
    deactivate P

    Note over FM, PO: 3. Review & Transmit
    FM->>FM: retrievePOs()
    FM->>PO: approve()
    PO->>PO: changeState("Approved")
    
    PO->>API: transmitPO(this)
    activate API
    
    alt Success
        API-->>PO: HTTP 200 OK
        PO->>PO: changeState("Sent")
        PO-->>FM: Show Success Message
    else Failure
        API-->>PO: TimeoutException
        PO->>PO: changeState("Failed")
        PO-->>FM: Show Error / Retry
    end
    deactivate API
    deactivate PO
```

### 4️⃣ UML Activity Diagram
```mermaid
flowchart TD
    classDef terminal fill:#333,stroke:#333,color:#fff,shape:circle;
    classDef decision fill:#fff9c4,stroke:#333,stroke-width:1.5px,color:#000;
    classDef process fill:#ffffff,stroke:#333,stroke-width:1.5px,color:#000;

    Start((Start)):::terminal
    End((End)):::terminal

    subgraph SysLane ["System (Event Listener)"]
        direction TB
        A1(["Update Product Stock"]):::process
        A2{"Stock <= Reorder Point?"}:::decision
        A3(["Calculate Required Quantity"]):::process
        A4(["Instantiate PO (State: Draft)"]):::process
    end

    subgraph FMLane ["Finance Manager"]
        direction TB
        F1(["Review Pending PO"]):::process
        F2{"Approval Decision?"}:::decision
        F3(["Set State: Rejected & Log Reason"]):::process
    end

    subgraph APILane ["Supplier API"]
        direction TB
        S1(["Transmit PO Data"]):::process
        S2{"Transmission Status?"}:::decision
        S3(["Set State: Failed - Timeout"]):::process
        S4(["Set State: Sent - HTTP 200"]):::process
    end

    Start --> A1
    A1 --> A2
    A2 -->|No - Stock Sufficient| End
    A2 -->|Yes - Low Stock| A3
    A3 --> A4
    A4 --> F1
    
    F1 --> F2
    F2 -->|Reject| F3
    F3 --> End
    
    F2 -->|Approve| S1
    S1 --> S2
    S2 -->|Error| S3
    S3 --> End
    S2 -->|Success| S4
