# Programming thoughts
My thoughts on programming/software engineering.

### 1. Method's statement should be on the same abstraction level.

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

### 2. Don't fight Optional in Java.
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

### 3. Methods bodies should be five lines long (or even less).
Think of method's size.
How can be large methods convinient? They can't.
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
Method (function or procedure) should do only one thing - wrap less abstract procedures invokations.
For instance, what is `person.addFriend(Person other)`? It is `if (other.subscribedTo(this)) { this.subscribeTo(other); }` (checks mutual subscription and subscribes one person to the other.
What is `person.subscribedTo(Person other)`? It is `person.subscriptions.contains(other)`. What is `subscribeTo`? It is `person.subscriptions.add(other)` then `person.save()`.
Abstract methods call a bit less abstract methods and they call even less abstract methods and so on. 
Very abstract methods (your application itself, business logic) should not contain low-level methods calls. It makes our software inflexible.
