# Module2 Project Worktime - ConcertService and Driver

## Learning Goals

- Spend some time working on your project.
- Implement a `ConcertService` class.
- Implement a `Driver` class as a command-line interface (CLI) application.

## Introduction

Now that we have learned more about working with multiple classes,
it is time to apply this new knowledge to our project!

You will continue modifying the module project that you previously forked from
[Module2 Project on Github](https://github.com/learn-co-curriculum/java-module2-project).


The `ConcertRepository` class encapsulates a collection of `Concert` objects
and provides methods to add a new concert, get a concert by index, and find
a concert by performer name.  However, the `ConcertRepository`
class does not provide business logic or input/output interaction
with the user.    This will be handled in two additional classes:

- The `Driver` class implements a command-line interface (CLI) to handle user input, repeatedly
  prompting the user with: "Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit:"
- The `Driver` class will delegate the request to a `ConcertService` object.
- The `ConcertService` class implements the business logic for the application.
    - Adding a concert:
        - Print a success message if the concert is added to the repository.
        - Print an error message if the repository already contains a concert for that performer.
        - Print an error message if the repository can't add a concert (array is full).
    - Display all concerts
        - Iterate and print each concert in the repository.
    - Purchase ticket:
        - Print a success message if the purchase was successful.
        - Print an error message if the repository does not contain a concert for the specified performer.
        - Print an error message if the repository contains the concert but the purchase is unsuccessful (no tickets available).
    - Add to waitlist:
        - Print a success message if the waitlist update was successful.
        - Print an error message if the repository does not contain a concert for the specified performer.



The `Driver` class will use the `ConcertService` class to perform the
actions requested by the user.  The `ConcertService` class in turn
uses the `ConcertRepository` class to manage the actual storage of `Concert`
objects in the array.

![uml module2 project](https://curriculum-content.s3.amazonaws.com/6676/java-multipleclasses/uml_module2_project.png)


We'll implement the `ConcertService` class first, then implement the CLI in the `Driver`.

## `ConcertService` class

(1) Edit the `ConcertService` class to add an instance variable to reference an instance of `ConcertRepository`
that stores a maximum of 3 concerts.

```java
private  ConcertRepository repository = new ConcertRepository(3);
```

(2) Edit the `ConcertService` class to add a method named `addConcert` that takes two
parameters: the performer name and the number of available tickets.  The method does not return a value.
- Call the `findByPerformer` method to test if a concert for that performer
  exists in the repository, and print "Unable to add concert" if one already does.
- If there is no existing concert for the performer, create a new `Concert` object
  and call the `add` method to add the new object to the repository.
    - Print "Added concert" if the `add` method returns `true`.
    - Print "Unable to add concert" if the `add` method returns `false`.

(3) Edit `ConcertServiceTest` to add two methods to test the new functionality:

```java
@Test
void addFull() {
    concertService.addConcert("Taylor Swift" , 100);
    concertService.addConcert("The Weeknd", 5000);
    concertService.addConcert("Harry Styles", 599);
    // Array size 3, can't add any more
    concertService.addConcert("Another Singer", 100);
    assertEquals("Added concert\n" +
                 "Added concert\n" +
                 "Added concert\n" +
                 "Unable to add concert",
                 outputStreamCaptor.toString().trim());
}

@Test
void duplicateAdd() {
    concertService.addConcert("Taylor Swift" , 100);
    concertService.addConcert("Taylor Swift" , 200);
    assertEquals("Added concert\n" +
                 "Unable to add concert",
                 outputStreamCaptor.toString().trim());
}
```

Run the Junit test and confirm your code.

(4) Edit the `ConcertService` class to
add a method named `displayConcerts` that takes no parameters and returns no value.
The method should call the repository's `get` method to retrieve and print each concert in the repository.

(5) Edit `ConcertServiceTest` to add two methods to test the new functionality:

```java
@Test
void displayEmpty() {
    concertService.displayConcerts();
    assertEquals("", outputStreamCaptor.toString().trim());
}

@Test
void displayNonEmpty() {
    concertService.addConcert("Taylor Swift" , 100);
    concertService.addConcert("The Weeknd", 5000);
    concertService.displayConcerts();
    assertEquals("Added concert\n" +
                 "Added concert\n" +
                 "Concert{artist='Taylor Swift', available=100, waitlist=0}\n" +
                 "Concert{artist='The Weeknd', available=5000, waitlist=0}",
                 outputStreamCaptor.toString().trim());
}
```

Run the Junit tests and confirm your code.

(6) Edit the `ConcertService` class to
add a method named `purchaseTicket`. The method takes one parameter, the performer name.
The method does not return a value.

- Call the `findByPerformer` method to test if a concert for that performer
  exists in the repository, and print "No concert for name", substituting the
  actual performer name, if there is no such concert.
- If there is a concert for the performer, call the `purchaseTicket` method on the given concert
  object.  Test the result returned from `purchaseTicket`.
    - Print "Ticket purchased" if the `purchaseTicket` method returns `true`.
    - Print "Ticket unavailable" if the `purchaseTicket` method returns `false`.

(7) Edit the `ConcertServiceTest` class to add Junit tests:

```java
@Test
void purchaseTicket() {
    concertService.addConcert("Taylor Swift" , 3);
    concertService.purchaseTicket("Taylor Swift");
    concertService.purchaseTicket("Taylor Swift");
    concertService.purchaseTicket("Taylor Swift");
    // sold out, ticket unavailable
    concertService.purchaseTicket("Taylor Swift");
    assertEquals("Added concert\n" +
                 "Ticket purchased\n" +
                 "Ticket purchased\n" +
                 "Ticket purchased\n" +
                 "Ticket unavailable",
            outputStreamCaptor.toString().trim());
}

@Test
void purchaseTicketUnknownArtist() {
    concertService.addConcert("Taylor Swift" , 1000);
    concertService.purchaseTicket("Unknown Singer");
    assertEquals("Added concert\n" +
                  "No concert for Unknown Singer",
            outputStreamCaptor.toString().trim());
}
```

Run the Junit tests and confirm your code.

(8) Edit the `ConcertService` class to add a method `addToWaitlist` that takes
one parameter, the performer name.  The method does not return a value.

- Call the `findByPerformer` method to test if a concert for that performer
  exists in the repository, and print "No concert for name", substituting the
  actual performer name, if there is no such concert.
- If there is a concert for the performer, call the `addToWaitlist` method on the given concert
  object and then print "Added to waitlist".

(9) Edit the `ConcertServiceTest` class to add Junit tests:

```java
@Test
void addToWaitlist() {
    concertService.addConcert("Taylor Swift" , 100);
    concertService.addConcert("The Weeknd", 5000);
    concertService.addToWaitlist("Taylor Swift");
    concertService.addToWaitlist("Taylor Swift");
    concertService.addToWaitlist("The Weeknd");
    // no concert
    concertService.addToWaitlist("Unknown Singer");

    assertEquals("Added concert\n" +
                 "Added concert\n" +
                 "Added to waitlist\n" +
                 "Added to waitlist\n" +
                 "Added to waitlist\n" +
                 "No concert for Unknown Singer",
                outputStreamCaptor.toString().trim());
}
```


## `Driver` class

Edit the `Driver` class to provide a CLI for managing concerts as shown:

```java
import java.util.InputMismatchException;
import java.util.Scanner;

public class Driver {

    private static ConcertService service = new ConcertService();

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String prompt = "Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: ";
        String action;

        // Prompt the user for an action
        System.out.println(prompt);
        action = scanner.nextLine();

        // Loop until the user enters q to quit
        while (!action.equals("q")) {
            switch (action) {
                case "a":
                    System.out.println("Enter the performer's name:");
                    String performer = scanner.nextLine();
                    try {
                        System.out.println("Enter the number of available tickets:");
                        int available = scanner.nextInt();
                        scanner.nextLine();  //consume the rest of the input
                        service.addConcert(performer, available);
                    }
                    catch (InputMismatchException e) {
                        System.out.println("Value was not an integer");
                    }
                case "d":
                    service.displayConcerts();
                    break;
                case "p":
                    System.out.println("Enter the performer's name:");
                    service.purchaseTicket( scanner.nextLine() );
                    break;
                case "w":
                    System.out.println("Enter the performer's name:");
                    service.addToWaitlist( scanner.nextLine() );
                    break;
                default:
                    System.out.println("Invalid choice: " + action);
            }

            // get the next action
            System.out.println(prompt);
            action = scanner.nextLine();
        }
    }

} 
```

Run the program and test the various actions.  For example:

```text
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
a
Enter the performer's name:
Taylor Swift
Enter the number of available tickets:
1000
Added concert
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
d
Concert{performer='Taylor Swift', available=1000, waitlist=0}
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
a
Enter the performer's name:
The Weeknd
Enter the number of available tickets:
2
Added concert
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
d
Concert{performer='Taylor Swift', available=1000, waitlist=0}
Concert{performer='The Weeknd', available=2, waitlist=0}
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
p
Enter the performer's name:
The Weeknd
Ticket purchased
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
d
Concert{performer='Taylor Swift', available=1000, waitlist=0}
Concert{performer='The Weeknd', available=1, waitlist=0}
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
p
Enter the performer's name:
The Weeknd
Ticket purchased
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
d
Concert{performer='Taylor Swift', available=1000, waitlist=0}
Concert{performer='The Weeknd', available=0, waitlist=0}
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
p
Enter the performer's name:
The Weeknd
Ticket unavailable
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
d
Concert{performer='Taylor Swift', available=1000, waitlist=0}
Concert{performer='The Weeknd', available=0, waitlist=0}
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
p
Enter the performer's name:
The band
No concert for The band
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
a
Enter the performer's name:
Taylor Swift
Enter the number of available tickets:
500
Unable to add concert
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
w
Enter the performer's name:
The Weeknd
Added to waitlist
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
d
Concert{performer='Taylor Swift', available=1000, waitlist=0}
Concert{performer='The Weeknd', available=0, waitlist=1}
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
w
Enter the performer's name:
Taylor Swift
Added to waitlist
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
d
Concert{performer='Taylor Swift', available=1000, waitlist=1}
Concert{performer='The Weeknd', available=0, waitlist=1}
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
w
Enter the performer's name:
The band
No concert for The band
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
d
Concert{performer='Taylor Swift', available=1000, waitlist=1}
Concert{performer='The Weeknd', available=0, waitlist=1}
Select an action: a=add concert, d=display all concerts, p=purchase ticket, w=add to waitlist, q=quit: 
q
```
