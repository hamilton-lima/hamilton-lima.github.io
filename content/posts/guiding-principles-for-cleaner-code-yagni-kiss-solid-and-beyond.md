---
title: 'Guiding Principles for Cleaner Code: YAGNI, KISS, SOLID, and Beyond'
date: Fri, 11 Oct 2024 12:56:42 +0000
draft: false
tags: ['clean-code', 'solid-principles', 'yagni', 'kiss', 'dependency-injection', 'software-engineering', 'code-quality', 'best-practices', 'testing', 'maintainability']
---

![Guiding Principles for Cleaner Code: YAGNI, KISS, SOLID, and Beyond](/images/2024/guiding-principles-for-cleaner-code-yagni-kiss-solid-and-beyond.jpg) 

"Any fool can write code that a computer can understand. Good programmers write code that humans can understand." - Martin Fowler [source](https://twitter.com/martinfowler/status/1067179181140844544#)

When you start your coding journey, the natural instinct is to write instructions as if you're talking to a person: "Hey computer, grab some eggs, butter, and flour, and let's bake a cake." Procedural it is—instruction after instruction, right? It seems easy to understand, but it’s actually not 😊. Here’s why: Consider the "bake a cake" example. It’s a simple set of instructions, but imagine if you also have to decorate the cake for a wedding and organize all the necessary items for the event. The complexity starts to grow, and soon enough, a simple list of instructions won’t be enough.

There are principles that can help in the journey of breaking down complex problems into code that other humans can understand. These principles aren’t meant to be followed blindly but serve as guidelines or tools in your toolbox for solving specific problems. That’s the tricky part of coding—deciding which principles to apply and when! When working in a team, if there are standards, I’d recommend following those standards first, then discussing ideas, not the other way around. After all, we’re building code for each other. We solve problems, and code is simply a tool.

There are plenty of great concepts and catchy phrases about coding quality. Some include:

- You aren't gonna need it (YAGNI)
- Dependency injection
- If it ain't broke, don't fix it
- SOLID principles
- Locality of behavior
- Keep it DRY (Don’t Repeat Yourself)
- Cyclomatic complexity
- Separation of concerns
- Keep it simple, stupid (KISS)
- Arrange, Act, Assert
- Worse is better
- Avoid premature optimization

Now, let me connect some concepts that are intentionally scattered in the list 😁. Let’s start with YAGNI—You Aren't Gonna Need It! This is powerful and is very much connected to the idea of "what about doing nothing?" But the natural instinct is to do something, fix the problem—what if there’s a tiger? 🐯☠️ The principles are here to help us mix instinct with rational decision-making. See [a love letter to ourselves](https://hamiltonlima.com/posts/love-letter-to-ourselves/).

### How to Apply YAGNI

Ask yourself, "Do we need this now?" Notice it's "Do *we* need…" and not "*I* need…". This encourages communication, agreements, and building *with* the team. For example, imagine you’re building a CLI tool that opens a CSV file and calculates the sum of a value column. Do you need to receive the filename as a parameter, or should the code look for a specific filename?

Consider this example:

```python
import csv

# Initialize sum variable
value_sum = 0

# Open the CSV file and read it
with open('sales.csv', mode='r') as file:
    csv_reader = csv.DictReader(file)
    # Iterate through each row in the CSV
    for row in csv_reader:
        # Add the value from the 'value' column to the sum
        value_sum += float(row['value'])

# Print the sum of the column
print(f"The sum of the 'value' column is: {value_sum}")
```

Now, the same code receiving the filename as a parameter and checking if the file exists:

```python
import csv
import sys
import os

# Initialize sum variable
value_sum = 0

# Get the filename from command line arguments
if len(sys.argv) < 2:
    print("Usage: python script.py <filename>")
    sys.exit(1)

file_name = sys.argv[1]

# Check if the file exists
if not os.path.exists(file_name):
    print(f"Error: The file '{file_name}' does not exist.")
    sys.exit(1)

# Open the CSV file and read it
with open(file_name, mode='r') as file:
    csv_reader = csv.DictReader(file)
    # Iterate through each row in the CSV
    for row in csv_reader:
        # Add the value from the 'value' column to the sum
        value_sum += float(row['value'])

# Print the sum of the column
print(f"The sum of the 'value' column is: {value_sum}")
```

Where does this lead? The second version seems more robust and error-proof, but the question is, "Do we need it now?" Maybe the answer could be:

- Yes, because most of this extra code is AI-generated, which might reduce potential human errors in the future.
- No, because this code won’t be used again anytime soon, and we have other priorities.

Both answers can be valid depending on the context. Regardless, the key is to be intentional and understand the side effects of each decision. The first version invests upfront with the risk of wasting time, while the second version postpones the investment, potentially taking more time in the future. Ultimately, the decision is about when to take the risk—now or later? This should be defined as a guideline for the team or project. It’s not simple, but the intention needs to be clear so the team can understand and follow the principle.

YAGNI can cover several other principles like "If it ain't broke, don't fix it," KISS (Keep It Simple, Stupid), "Worse is better," and "Avoid premature optimization."

### Does YAGNI Relate to These Principles?

- Dependency injection
- SOLID principles
- Locality of behavior

When you dive into the SOLID principles, you see a deep connection with the creation of class hierarchies, which can become complicated quickly—especially when multiple abstraction levels are involved, like Person -> Employee -> Intern. Or when creating nicely organized interfaces as the Dependency Inversion Principle (the "D" in SOLID) suggests. Group behavior in small interfaces, but consider YAGNI. Let the abstraction grow as the team and codebase needs evolve.

SOLID principles are pretty cool. Take the "S" and "O" (Single Responsibility and Open/Closed) principles as examples—they’re well described by dependency injection. When you pass other classes as parameters to mutate a class, you’re keeping responsibilities distinct and enabling future extension by injecting different behaviors. What does that mean? Let’s look at an example:

```java
abstract class Payment {
    public abstract void processPayment(double amount);
}

class CreditCardPayment extends Payment {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing credit card payment: $" + amount);
    }
}

class PayPalPayment extends Payment {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing PayPal payment: $" + amount);
    }
}

class PaymentProcessor {
    public void process(Payment payment, double amount) {
        payment.processPayment(amount);
    }
}

public class LiskovSubstitutionExample {
    public static void main(String[] args) {
        PaymentProcessor processor = new PaymentProcessor();

        Payment creditCard = new CreditCardPayment();
        Payment paypal = new PayPalPayment();

        processor.process(creditCard, 100.0);  // Correct behavior
        processor.process(paypal, 150.0);      // Correct behavior
    }
}
```

The `PaymentProcessor` receives `Payment` as a parameter, applying the dependency injection concept. `PaymentProcessor` doesn’t know, or need to know, the details—whether it's a credit card or PayPal payment 😊.

Dependency injection helps understand the Single Responsibility and Open/Closed principles. What about Liskov? Named after [Barbara Liskov](https://en.wikipedia.org/wiki/Barbara_Liskov), the principle states that subclasses should be interchangeable with the base class. This example demonstrates that both payment types can be used in the `process` method.

The Interface Segregation Principle is about keeping interfaces as small as possible. It’s a constant discovery process—you might start with a more "rich" interface so the Dependency Inversion Principle can create the right dependencies, and as understanding evolves, the interfaces can be broken down. Avoid breaking down interfaces too much, too early—remember YAGNI.

### What if We Don’t Want to Add SOLID Abstractions?

Here’s how it could look:

```java
class PaymentProcessor {
    public void process(String paymentType, double amount) {
        if (paymentType.equals("CreditCard")) {
            processCreditCard(amount);
            return;
        }
        if (paymentType.equals("PayPal")) {
            processPayPal(amount);
            return;
        }
        throw new IllegalArgumentException("Unknown payment type: " + paymentType);
    }

    private void processCreditCard(double amount) {
        System.out.println("Processing credit card payment: $" + amount);
    }

    private void processPayPal(double amount) {
        System.out.println("Processing PayPal payment: $" + amount);
    }
}

public class LocalityOfBehaviorExample {
    public static void main(String[] args) {
        PaymentProcessor processor = new PaymentProcessor();

        processor.process("CreditCard", 100.0);  // Correct behavior
        processor.process("PayPal", 150.0);      // Correct behavior
    }
}
```

Some might say this is worse, while others might say it’s more clear—it feels more comfortable to have all elements in the same class. The discussion returns to why you do it. This approach might work if you’re working alone or there’s no intention of expanding or reusing the code. Most of the time, applying the SOLID principles is a safer bet, especially if you value testing, as abstractions make applying the Arrange, Act, Assert principle easier in your tests.

Tell me—are SOLID principles the right approach, or do you prefer the path of composition?
