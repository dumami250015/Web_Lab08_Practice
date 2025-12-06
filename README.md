# LAB 08 PRACTICE REPORT - Customer REST API

## Application Code Flow

### 1. GET All Customers Flow
This flow retrieves the complete list of all customers in the system.

1. **HTTP Request**: Client sends `GET http://localhost:8080/api/customers`.
2. **Controller**: `CustomerRestController.getAllCustomers()` receives the request.
3. **Service Layer**: Controller calls `CustomerService.getAllCustomers()`.
4. **Repository**: Service calls `CustomerRepository.findAll()` to fetch all customer records from the database.
5. **DTO Conversion**: Each `Customer` entity is converted to `CustomerResponseDTO` using `convertToResponseDTO()`.
6. **Response**: Returns `200 OK` with a JSON array of customer objects.

**Example Response:**
```json
[
  {
    "id": 1,
    "customerCode": "C001",
    "fullName": "John Doe",
    "email": "john@example.com",
    "phone": "1234567890",
    "address": "123 Main St",
    "status": "ACTIVE",
    "createdAt": "2025-12-06T10:00:00"
  },
  ...
]
```

### 2. GET Customer by ID Flow
This flow retrieves a single customer by their unique identifier.

1. **HTTP Request**: Client sends `GET http://localhost:8080/api/customers/{id}`.
2. **Controller**: `CustomerRestController.getCustomerById(Long id)` extracts the ID from the URL path.
3. **Service Layer**: Controller calls `CustomerService.getCustomerById(id)`.
4. **Repository**: Service calls `CustomerRepository.findById(id)`.
5. **Exception Handling**: 
   * **Success**: If found, the entity is converted to `CustomerResponseDTO`.
   * **Failure**: If not found, throws `ResourceNotFoundException` (handled by Global Exception Handler).
6. **Response**: Returns `200 OK` with the customer object, or `404 Not Found` if the customer doesn't exist.

**Success Response (200):**
```json
{
  "id": 1,
  "customerCode": "C001",
  "fullName": "John Doe",
  "email": "john@example.com",
  "phone": "1234567890",
  "address": "123 Main St",
  "status": "ACTIVE",
  "createdAt": "2025-12-06T10:00:00"
}
```

### 3. POST Create Customer Flow
This flow creates a new customer in the system with validation.

1. **HTTP Request**: Client sends `POST http://localhost:8080/api/customers` with JSON body.
2. **Validation**: Spring's `@Valid` annotation triggers validation on `CustomerRequestDTO`:
   * Customer code format: Must start with 'C' followed by numbers (e.g., C001).
   * Email format validation.
   * Phone number: Must be 10-20 digits.
   * All required fields must be present.
3. **Controller**: `CustomerRestController.createCustomer(@Valid CustomerRequestDTO)` receives validated data.
4. **Service Layer**: Controller calls `CustomerService.createCustomer(requestDTO)`.
5. **Duplicate Check**: Service validates:
   * `CustomerRepository.existsByCustomerCode()` - Checks if customer code already exists.
   * `CustomerRepository.existsByEmail()` - Checks if email already exists.
   * If duplicates found, throws `DuplicateResourceException`.
6. **Entity Conversion**: Converts `CustomerRequestDTO` to `Customer` entity using `convertToEntity()`.
7. **Database Save**: `CustomerRepository.save(customer)` persists the new customer.
8. **Response Conversion**: Saved entity is converted to `CustomerResponseDTO`.
9. **Response**: Returns `201 Created` with the newly created customer object.

**Request Body:**
```json
{
  "customerCode": "C006",
  "fullName": "David Miller",
  "email": "david.miller@example.com",
  "phone": "1555010600",
  "address": "999 Broadway, Seattle, WA 98101"
}
```

**Success Response (201):**
```json
{
  "id": 6,
  "customerCode": "C006",
  "fullName": "David Miller",
  "email": "david.miller@example.com",
  "phone": "1555010600",
  "address": "999 Broadway, Seattle, WA 98101",
  "status": "ACTIVE",
  "createdAt": "2025-12-06T10:21:00"
}
```

### 4. PUT Update Customer Flow
This flow updates an existing customer's information.

1. **HTTP Request**: Client sends `PUT http://localhost:8080/api/customers/{id}` with JSON body.
2. **Validation**: `@Valid` annotation validates the `CustomerRequestDTO` input.
3. **Controller**: `CustomerRestController.updateCustomer(Long id, @Valid CustomerRequestDTO)` receives the request.
4. **Service Layer**: Controller calls `CustomerService.updateCustomer(id, requestDTO)`.
5. **Existence Check**: Service calls `CustomerRepository.findById(id)`:
   * **Not Found**: Throws `ResourceNotFoundException`.
   * **Found**: Retrieves existing customer entity.
6. **Email Duplicate Check**: If email is being changed, validates that the new email doesn't already exist:
   * Calls `CustomerRepository.existsByEmail()`.
   * If duplicate found, throws `DuplicateResourceException`.
7. **Update Fields**: Updates mutable fields (fullName, email, phone, address). Note: `customerCode` is immutable.
8. **Database Save**: `CustomerRepository.save(existingCustomer)` persists the changes.
9. **Response Conversion**: Updated entity is converted to `CustomerResponseDTO`.
10. **Response**: Returns `200 OK` with the updated customer object.

**Request Body:**
```json
{
  "customerCode": "C006",
  "fullName": "David M. Miller",
  "email": "david.m.miller@example.com",
  "phone": "1555010601",
  "address": "1000 Broadway, Seattle, WA 98101"
}
```

**Success Response (200):**
```json
{
  "id": 6,
  "customerCode": "C006",
  "fullName": "David M. Miller",
  "email": "david.m.miller@example.com",
  "phone": "1555010601",
  "address": "1000 Broadway, Seattle, WA 98101",
  "status": "ACTIVE",
  "createdAt": "2025-12-06T10:21:00"
}
```

### 5. DELETE Customer Flow
This flow removes a customer from the system.

1. **HTTP Request**: Client sends `DELETE http://localhost:8080/api/customers/{id}`.
2. **Controller**: `CustomerRestController.deleteCustomer(Long id)` extracts the ID.
3. **Service Layer**: Controller calls `CustomerService.deleteCustomer(id)`.
4. **Existence Check**: Service calls `CustomerRepository.existsById(id)`:
   * **Not Found**: Throws `ResourceNotFoundException`.
   * **Found**: Proceeds with deletion.
5. **Database Delete**: `CustomerRepository.deleteById(id)` removes the customer.
6. **Response**: Returns `200 OK` with a success message.

**Success Response (200):**
```json
{
  "message": "Customer deleted successfully"
}
```

### 6. Search Customers Flow
This flow searches for customers by keyword across multiple fields.

1. **HTTP Request**: Client sends `GET http://localhost:8080/api/customers/search?keyword={keyword}`.
2. **Controller**: `CustomerRestController.searchCustomers(@RequestParam String keyword)` extracts the query parameter.
3. **Service Layer**: Controller calls `CustomerService.searchCustomers(keyword)`.
4. **Repository**: Service calls `CustomerRepository.searchCustomers(keyword)`:
   * Uses custom JPQL query to search across `fullName`, `email`, and `customerCode` fields.
   * Performs case-insensitive partial matching using `LIKE '%keyword%'`.
5. **DTO Conversion**: Each matching `Customer` entity is converted to `CustomerResponseDTO`.
6. **Response**: Returns `200 OK` with an array of matching customers (empty array if no matches).

**Example Request:** `GET /api/customers/search?keyword=john`

**Response (200):**
```json
[
  {
    "id": 1,
    "customerCode": "C001",
    "fullName": "John Doe",
    "email": "john@example.com",
    ...
  }
]
```

### 7. Get Customers by Status Flow
This flow retrieves all customers with a specific status (ACTIVE/INACTIVE).

1. **HTTP Request**: Client sends `GET http://localhost:8080/api/customers/status/{status}`.
2. **Controller**: `CustomerRestController.getCustomersByStatus(@PathVariable String status)` extracts the status.
3. **Service Layer**: Controller calls `CustomerService.getCustomersByStatus(status)`.
4. **Repository**: Service calls `CustomerRepository.findByStatus(status)`.
5. **DTO Conversion**: Each matching `Customer` entity is converted to `CustomerResponseDTO`.
6. **Response**: Returns `200 OK` with an array of customers with the specified status.

**Example Request:** `GET /api/customers/status/ACTIVE`

**Response (200):**
```json
[
  {
    "id": 1,
    "customerCode": "C001",
    "status": "ACTIVE",
    ...
  },
  ...
]
```

## Error Handling Flows

### 8. Validation Error Flow (400 Bad Request)
This flow handles invalid input data during CREATE or UPDATE operations.

1. **Trigger**: Client sends invalid data (e.g., missing required fields, invalid format).
2. **Validation**: Spring's `@Valid` annotation triggers validation on the DTO:
   * Validates `@NotBlank`, `@Email`, `@Pattern`, `@Size` constraints.
   * If validation fails, throws `MethodArgumentNotValidException`.
3. **Exception Handler**: `GlobalExceptionHandler.handleValidationException()` catches the exception:
   * Extracts all validation errors from `BindingResult`.
   * Creates `ErrorResponseDTO` with status `400` and detailed error messages.
4. **Response**: Returns `400 Bad Request` with structured error information.

**Example Invalid Request:**
```json
{
  "customerCode": "C006",
  "fullName": "D",
  "email": "invalid-email",
  "phone": "+1555",
  "address": "999 Broadway"
}
```

**Error Response (400):**
```json
{
  "timestamp": "2025-12-06T10:21:10.449",
  "status": 400,
  "error": "Validation Failed",
  "message": "Invalid input data",
  "path": "/api/customers",
  "details": [
    "fullName: Name must be 2-100 characters",
    "email: Invalid email format",
    "phone: Invalid phone number format"
  ]
}
```

### 9. Resource Not Found Flow (404 Not Found)
This flow handles requests for non-existent resources.

1. **Trigger**: Client requests a customer with an ID that doesn't exist (GET, PUT, or DELETE).
2. **Repository Check**: `CustomerRepository.findById()` or `existsById()` returns empty.
3. **Exception Thrown**: Service layer throws `ResourceNotFoundException` with descriptive message.
4. **Exception Handler**: `GlobalExceptionHandler.handleResourceNotFoundException()` catches the exception:
   * Creates `ErrorResponseDTO` with status `404`.
5. **Response**: Returns `404 Not Found` with error details.

**Example Request:** `GET /api/customers/999` (ID doesn't exist)

**Error Response (404):**
```json
{
  "timestamp": "2025-12-06T10:25:00.123",
  "status": 404,
  "error": "Not Found",
  "message": "Customer not found with id: 999",
  "path": "/api/customers/999"
}
```

### 10. Duplicate Resource Flow (409 Conflict)
This flow handles attempts to create or update with duplicate unique fields.

1. **Trigger**: Client attempts to:
   * Create a customer with existing `customerCode` or `email`.
   * Update a customer's email to one that already exists.
2. **Duplicate Check**: Service layer calls:
   * `CustomerRepository.existsByCustomerCode()` - Returns `true` if code exists.
   * `CustomerRepository.existsByEmail()` - Returns `true` if email exists.
3. **Exception Thrown**: Service throws `DuplicateResourceException` with specific message.
4. **Exception Handler**: `GlobalExceptionHandler.handleDuplicateResourceException()` catches the exception:
   * Creates `ErrorResponseDTO` with status `409`.
5. **Response**: Returns `409 Conflict` with error details.

**Example Request:** Creating customer with existing email
```json
{
  "customerCode": "C007",
  "fullName": "Jane Smith",
  "email": "john@example.com",
  ...
}
```

**Error Response (409):**
```json
{
  "timestamp": "2025-12-06T10:30:00.456",
  "status": 409,
  "error": "Conflict",
  "message": "Email already exists: john@example.com",
  "path": "/api/customers"
}
```

## Architecture Components

### Controller Layer (`CustomerRestController`)
* Handles HTTP requests and responses.
* Performs request/response mapping using `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`.
* Validates input using `@Valid` annotation.
* Returns appropriate HTTP status codes.

### Service Layer (`CustomerService` & `CustomerServiceImpl`)
* Contains business logic and validation rules.
* Performs duplicate checks before CREATE/UPDATE operations.
* Handles DTO to Entity conversion and vice versa.
* Throws custom exceptions for error scenarios.

### Repository Layer (`CustomerRepository`)
* Extends `JpaRepository` for database operations.
* Provides custom query methods (`findByStatus`, `existsByEmail`, etc.).
* Uses JPQL for complex search queries.

### Exception Handling (`GlobalExceptionHandler`)
* Centralized error handling using `@RestControllerAdvice`.
* Catches and transforms exceptions into standardized error responses.
* Handles:
  * `ResourceNotFoundException` → 404
  * `DuplicateResourceException` → 409
  * `MethodArgumentNotValidException` → 400
  * Generic `Exception` → 500

### DTO Layer
* **`CustomerRequestDTO`**: Input validation and data transfer for CREATE/UPDATE.
* **`CustomerResponseDTO`**: Output data structure for API responses.
* **`ErrorResponseDTO`**: Standardized error response format.

### Entity Layer
* **`Customer`**: JPA entity mapping to the `customers` table.
* **`CustomerStatus`**: Enum defining customer statuses (ACTIVE, INACTIVE).
