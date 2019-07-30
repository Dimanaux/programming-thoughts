# Programming thoughts
My thoughts on programming/software engineering.

## Contents
0. [Method's statement should be on the same abstraction level.](#methods-statement-should-be-on-the-same-abstraction-level)
0. [Don't fight Optional in Java.](#dont-fight-optional-in-java)
0. [Methods bodies should be five lines long (or even less).](#methods-bodies-should-be-five-lines-long-or-even-less)

## Method's statement should be on the same abstraction level.

If we write method body and we get a huge one that means we build wrong abstraction.
Our procedures used in this body should be on the same abstraction level.
```java
public void sendMessage(Message message, List<User> recipients) {
    for (User u : recipients) {
        u.setMessage(message);
        userRepository.save(u);
        message.addRecipient(u);
        messageRepository.save(message);
    }
    notifyRecipients(recipients);
} // Wrong!   
```
Here we can see that you use `notifyRecipients` function in the same method where `for` loop is used, in `for` loop we save our users and message. It is very unreadable since we have to understand something terrible and incomprehensible from this large `for` statement.
Imagine we have this method implementation instead:
```java
public void sendMessage(Message message, List<User> recipients) {
    message.sendAwayTo(recipients);
    notifyRecipients(recipients);
}
```
Now we perceive the meaning of this procedure much more clear, because names of functions tell us what the main idea of our method consists of.

## Don't fight Optional in Java.

Many programmers don't know much about Optional in Java and they even don't want to understand it and use it in their purposes.
```java
public User authenticate(HttpServletRequest req) {
    String username = req.getParameter("username");
    String password = securityService.hashPassword(
            req.getParameter("password")
    );

    User user = userDao.getByUsername(username);
    if (user != null && user.getPassword().equals(password)) {
        return user;
    } else {
        return null;
    }
}
```
Look at my code I wrote when I didn't know that userDao can return Optional<User> instead of just User instance.
You should not avoid using Optional, you can interact with it like it is almost a User instance.
```java
public Optional<Account> authenticate(String username, String password) {
    Optional<Account> account = accountRepo.findByUsername(username);
    return account.filter(u -> encoder.matches(password, u.getPassword()));
}
```
`Optional<User>` is not some odd object that makes you call `.get()` method and handle exceptions or check if it is null or something like this. 
It is your OBJECT ITSELF, but it may not present. You can invoke all its methods!!! like this:
```java
Optional<Account> a = accountRepository.getFirst();
a.map(Account::email) // Optional<String>
        .map(messageService::sendNotificationEmail) // sends email if user was found
        // Optional<EmailMessage>
        .orElseThrow(AccountNotFoundException::new); // notifies us that there were no user found
// OR
map.put("fullName", a.map(Account::getFullName).orElse("User not found"));
```

## Methods bodies should be five lines long (or even less).

Think of method's size.
How can large methods be convenient? They can't.
If we write 5 lines of code in method we can clearly understand what it does.
It is easy to debug.
I have met 300-lined methods, invoking other huge methods and so on.
It is very hard to understand what is happening there.
They check something modyfiyng objects' states at the same time.
They create a big mess.
Also classes should be very small and contain a few methods.

And the best method is one-lined method. It looks like:
```java
class UserWithVacation {
    private final User user;
    
    UserWithVacation(User user) {
        this.user = user;
    }
    
    public Hours vacationBalance() {
        return Hours.of(user.vacatoinDaysLeft().size()).times(user.workHoursPerDay());
    }
}
```
Here is a sign of bad method that does 2 things or more: we insert empty line in it.
What is 2 things? Sounds abstract. It does.
Method (function or procedure) should do only one thing - wrap less abstract procedures invocations.

For instance, what is `person.addFriend(Person other)`? It is `if (other.subscribedTo(this)) { this.subscribeTo(other); }` (checks mutual subscription and subscribes one person to the other.

What is `person.subscribedTo(Person other)`? It is `person.subscriptions.contains(other)`. What is `subscribeTo`? It is `person.subscriptions.add(other)` then `person.save()`.

Abstract methods call a bit less abstract methods and they call even less abstract methods and so on. 
Very abstract methods (your application itself, business logic) should not contain low-level methods calls. It makes our software inflexible.

## We don't know OOP.

What is Object-oriented programming?
It is a paradigm where we use objects.
Objects are like small machines with its inner state (position of all of its gears) and functions (buttons and livers).
We can't intervene in it and move the gears, because we can easily break it.

Also rotating a gear in our machine doesn't make any sense. We won't know what will happen.
But we can interact with the machine using its buttons. All the buttons together form an interface.
Also with interface we abstract its internal organization and we don't need to be puzzled with question "Which gear do we need to rotate to make the machine print our document?"

Java is positioned as an object-oriented programming language.
Now look to typical Java web application.
```
application
├─── controller
├─── model
│    └ User.java
├─── repository
└─── service
     └ UserService.java
```
Entity.java (access modifiers are omitted).
```java
class User {
    String name;
    int age;
    
    User() {}
    String getName() {
        return name;
    }
    int getAge() {
        return age;
    }
    void setName(String name) {
        this.name = name;
    }
    void setAge(int age) {
        this.age = age;
    }
}
```
Huh! Looks like a structure in procedural programming language C.

UserService.java
```java
class UserService {
    UserRepository userRepository;
    
    boolean isAdult(User user) {
        return user.getAge() >= 18;
    }
    String firstName(User user) {
        return user.getName().split(" ")[0];
    }
    void updateName(User user, String name) {
        user.setName(name);
        userRepository.persist(user);
    }
    void updateAge(User user, int age) {
        if (age < 0 || age > 256) {
            throw new AgeException("Invalid age: " + age);
        }
        user.setAge(age);
        userRepository.persist(user);
    }
}
```

What are gears, buttons for User? For UserService? Nothing. User creates no abstraction.

We separeted User's functionality. Now we have only data structures and some functions for it.

We still can break User like this: `user.setAge(-1); userRepository.persist(user);`

It is not an object-oriented program.

```C
struct User { 
   char* name;
   int age;
};

int isAdult(struct User* user) {
    return *(user->age) >= 18;
}
char* firstName(struct User* user) {
    char* lastName = strchr(user->name, ' ');
    int firstNameLength = lastName - user->name;
    char* firstName = (char*)malloc((firstNameLength + 1) * sizeof(char));
    strncpy(firstName, user->name, firstNameLength);
    return firstName;
}
void updateName(struct User* user, char* name) {
    free(user->name);
    user->name = name;
    userRepository->persist(user);
}
int updateAge(struct User* user, int age) {
    if (age < 0 || age > 256) {
        return -1;
    }
    user->age = age;
    userRepository->persist(user);
    return 0;
}
```

