# SQLProjects
In this reporsitory I will showcase all my projects with the hopes of obtaining feedback in order to improve my coding skills

My first of many projects was the "Library Management System". In this project I develop a Library Management System that would allow a normal Library to manage their book inventory.
For that I have created and developed a database to provide a quick and convenient Library catalog.
Please notice that all data inputed in this database was created randomly apart from Book Titles and it’s publication dates. 

I would like to start by explaining the creation of tables. 
Innitially we would start off with 4 main tables. These would be related to data that we would want to store for future references.
The tables are:
  Customer - With the following fields
      CustomerID PK
      Name_Customer
      Address_Customer
      Phone_Customer
      Email_Customer 
      (notice that constraints are given for email and phone format)
  Librarian - So that we can store data on our employees - With the following fields
      EmployeeID PK
      Name_Employee
      Phone_Employee
      Address_Employee
      (notice that constraints are given for email and phone format)
  Books - with the following fields
      BookID PK
      ISBN
      Title
      Author
      Publication_Date
      Genre
      Virtual (whether it is an E-Book or real book) This is in a BIT data Type therefore 1 = TRUE and 0 = FALSE
      Quantity
      (notice that constraints are given for ISBN, Quantity and Virtual)
  Borrow - Which will store the record of rents that were made
      BorrowID PK
      CustomerID FK
      BookID FK
      Rent_Date
      Due_Date (by default rent have a 7 day period
      EmployeeID FK
      Returned (whether it has been returned or not) This is in a BIT data Type therefore 1 = TRUE and 0 = FALSE.
      
  After implementing the Tables and populating it with data I prepared some normal procedures that would be likely to happen in a real life situation:
  
  First procedure to be created was to be used when a client wanted to request a book.
CREATE PROCEDURE SPCustomerBorrows
	(@BookID as int
	,@CustomerID as int
	,@employeeID as int
	)
AS
	BEGIN
		IF (SELECT quantity from Books where BookID=@BookID) = 0
		BEGIN
			RAISERROR('This book is out of order',16,2)
		END
		ELSE
		BEGIN
			INSERT INTO Borrow (CustomerID,BookID,EmployeeID) VALUES
			(@customerID,@BookID,@employeeID)
			UPDATE Books set Quantity=Quantity-1 WHERE BookID = @BookID
		END		
	END
In my belief it is always important to have information both about the customer but also the employee. This Procedure allows us not only to add an extra row to our Borrow Table, but also return an Error message if the book is out of order and if the customer in fact requests the book to update the quantity of books available in our Library.
The return date is also automatically calculated being that our Library has a 7 day period of rental. (this is done by setting default values when creating the table).
If the manager of the Library would like to be more agile in the period that it grants customers to rent their books, we can add an extra line to this Procedure to also include a certain period, which would look like this. (the changes are highlighted in Yellow)
ALTER PROCEDURE SPCustomerBorrows
	(@BookID as int
	,@CustomerID as int
	,@employeeID as int
	,@RentalPeriod as int = 7
	)
AS
	BEGIN
		IF (SELECT quantity from Books where BookID=@BookID) = 0
		BEGIN
			RAISERROR('This book is out of order',16,2)
		END
		ELSE
		BEGIN
			INSERT INTO Borrow (CustomerID,BookID,EmployeeID,Due_Date) VALUES
			(@customerID,@BookID,@employeeID,(GETDATE()+@rentalperiod))
			UPDATE Books set Quantity=Quantity-1 WHERE BookID = @BookID
		END		
	END
Let us see an example of this procedure in action. Let’s imagine that our customer Barry Philips with the customer ID of 8 would like to borrow our Book called “A Gentleman in Moscow: A Novel” with the BookID of 8 and did this procedure with our Librarian Patricia Breen with the Employeed ID of 3 and he would like to borrow the book for 15 days. The simple command to use would be the following:
EXEC SPCustomerBorrows @customerid=8, @bookid = 12, @employeeID = 3, @RentalPeriod=15

 
Now we will move on to the next action that may happen in our Library which is the return of the book from our customer and eventually calculating any late delivery fee. Let us suppose that the fee is 1,50€ per day that it is delayed.
CREATE PROCEDURE SPBookReturn
	(@BookID as int
	,@customerID as int
	)
AS
	BEGIN
		IF (select due_date from Borrow where CustomerID=@customerID AND BookID=@BookID) < getdate()
			BEGIN 
				DECLARE @DELAY AS INT
				SET @DELAY = DATEDIFF(DAY,(SELECT due_date FROM Borrow WHERE CustomerID=@customerID AND BookID=@BookID),GETDATE())
				PRINT 'This book was delivered with a delay of' + @delay + 'days, which amounts to a delay of of'+ (@delay * 1.5) + '€'
				UPDATE Borrow SET Returned = 1 where CustomerID=@customerID
			END
		ELSE
			BEGIN
				UPDATE Borrow SET Returned = 1 WHERE CustomerID=@customerID AND BookID=@BookID
			END
	END
If Mr. Barry Philips were to deliver his book this would be the command line to execute:
EXECUTE spbookreturn @bookid =12, @customerid=8

Noticed that the “Return” column has changed status to 1 (which is equal to TRUE)

  Let us see some simple commands that we could perform with this Database that would have real world application:
    Checking which books are overdue:
      SELECT * FROM borrow WHERE Due_Date<GETDATE()
    Seeing which Drama books are available to rent:
      SELECT * FROM Books WHERE genre like 'Drama' and Quantity>0
    Checking which Virtual books are available:
      SELECT * FROM BOOKS WHERE Virtual = 1 
      In this case we would not have to check the quantity because E-books have unlimited downloads.
    Seeing which genres were available in our Library
      SELECT DISTINCT genre FROM BOOKS


  

