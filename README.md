# Python: Everything is an Object - Understanding What Really Happens

![Python Objects Banner](https://raw.githubusercontent.com/rpokman/holbertonschool-higher_level_programming/main/images/Python%20-%20Everything%20is%20object.jpg)

## Introduction

Have you ever had a Python variable mysteriously change its value when you didn't touch it? Or wondered why modifying a list inside a function affects the original list, but doing the same with a number doesn't? These aren't bugs—they're features of how Python handles objects. Everything in Python is an object, and understanding this simple truth will save you from countless frustrating debugging sessions. This post will help you understand why your code behaves the way it does, using real-world examples you'll encounter every day.

## id and type

Think of Python objects like houses. Every house has an address (its ID) and contents (its value). When you create a variable, you're creating a reference to that house's address.

Understanding the difference between **identity** (`is`) and **equality** (`==`) is crucial. The `==` operator compares values, while `is` compares identities:

```python
>>> a = [1, 2, 3]
>>> b = [1, 2, 3]
>>> a == b  # Same values?
True
>>> a is b  # Same object?
False
```

Here, `a` and `b` are like two different houses with the same furniture inside. They have identical contents (`==` is True), but they're at different addresses (`is` is False).

However, Python is smart about saving memory. For small numbers, it reuses the same object:

```python
>>> x = 89
>>> y = 89
>>> x is y  # Same object!
True
```

This works because Python caches small integers for efficiency—like having one community mailbox instead of individual ones for each house.

## mutable objects

Imagine you and your roommate sharing a shopping list on the fridge. When one person adds an item, both see the change because it's the **same list**. That's how mutable objects work in Python—lists, dictionaries, and sets can be modified, and everyone pointing to them sees the changes.

**The Problem:**

```python
>>> shopping_list = ["milk", "bread", "eggs"]
>>> roommate_list = shopping_list  # Both point to the same list
>>> roommate_list.append("coffee")
>>> print(shopping_list)
['milk', 'bread', 'eggs', 'coffee']  # Your list changed too!
```

This is the most common source of bugs for Python beginners. When you write `roommate_list = shopping_list`, you're **not** making a copy—you're just giving the same list another name.

**Real-world scenario:** You're building a web app and accidentally modify a shared user list:

```python
default_permissions = ["read", "write"]

def create_user(name):
    user = {"name": name, "permissions": default_permissions}
    return user

admin = create_user("Alice")
admin["permissions"].append("admin")  # Oops!

regular_user = create_user("Bob")
print(regular_user["permissions"])
# ['read', 'write', 'admin']  # Bob is now an admin too!
```

**The Solution:** Make a copy when you need independent lists:

```python
>>> my_list = ["milk", "bread"]
>>> their_list = my_list[:]  # Creates a new list
>>> their_list.append("coffee")
>>> print(my_list)
['milk', 'bread']  # Unchanged!
```

## immutable objects

Now imagine your name tag at a conference. Once printed, you can't change it—you'd need to print a new one. That's how immutable objects work: integers, strings, and tuples can't be modified. Any "change" creates a brand new object.

**Example:**

```python
>>> name = "John"
>>> name = name + " Doe"  # Creates a NEW string
```

What looks like modifying `name` is actually creating a completely new string "John Doe" and pointing `name` to it. The original "John" string still exists (until Python's garbage collector cleans it up).

**Why it matters:** This is why this doesn't work like you might expect:

```python
>>> greeting = "Hello"
>>> copy = greeting
>>> greeting = greeting + " World"
>>> print(copy)
Hello  # Still the original value
```

The key takeaway: **You can't accidentally modify immutable objects**, which makes them safer to pass around in your code.

**Tuple quirk:** The comma makes the tuple, not the parentheses!

```python
>>> single = (1)     # This is just a number
>>> single_tuple = (1,)  # This is a tuple with one item
```

## why does it matter and how differently does Python treat mutable and immutable objects

Understanding mutability prevents bugs and helps you write better code. Python treats these two categories fundamentally differently:

**1. Dictionary keys must be immutable:**

```python
# This works
user_data = {("John", "Doe"): "john@email.com"}

# This doesn't
user_data = {["John", "Doe"]: "john@email.com"}  # Error!
```

**2. The `+=` operator behaves differently:**

```python
# With lists (mutable) - modifies in place
>>> scores = [10, 20]
>>> backup = scores
>>> scores += [30]
>>> print(backup)
[10, 20, 30]  # Backup changed too!

# With strings (immutable) - creates new object
>>> name = "John"
>>> backup = name
>>> name += " Doe"
>>> print(backup)
John  # Backup unchanged
```

**3. Python caches immutable objects for efficiency:**

Small integers and strings are reused to save memory, but mutable objects never are. This is why two separate lists with the same values aren't the same object, but two variables with the value `89` are.

## how arguments are passed to functions and what does that imply for mutable and immutable objects

Think of function parameters like borrowing items from a friend. What happens depends on what you borrowed:

**With immutable objects (numbers, strings, tuples):**
You can write on it, but your friend's original stays the same.

```python
def change_age(age):
    age += 1  # Creates a new number
    print(f"Inside function: {age}")

my_age = 25
change_age(my_age)
# Inside function: 26
print(f"Outside function: {my_age}")
# Outside function: 25  # Unchanged!
```

**With mutable objects (lists, dicts, sets):**
Your edits appear for everyone.

```python
def add_item(shopping_list):
    shopping_list.append("cookies")  # Modifies the original

my_list = ["milk", "bread"]
add_item(my_list)
print(my_list)
# ['milk', 'bread', 'cookies']  # Changed!
```

**The trap - trying to replace the whole thing:**

```python
def replace_list(old_list):
    old_list = ["completely", "new", "list"]  # Only affects local variable!

my_list = ["original"]
replace_list(my_list)
print(my_list)
# ['original']  # Unchanged because we only reassigned the local variable
```

**The dangerous default parameter bug:**

```python
# DON'T DO THIS
def add_student(name, class_list=[]):
    class_list.append(name)
    return class_list

math_class = add_student("Alice")  # ['Alice']
english_class = add_student("Bob")  # ['Alice', 'Bob'] - Oops!

# DO THIS INSTEAD
def add_student(name, class_list=None):
    if class_list is None:
        class_list = []
    class_list.append(name)
    return class_list
```

This happens because the default list is created once when the function is defined, not each time it's called. Since it's mutable, it keeps accumulating values!

## Conclusion: Three Simple Rules to Remember

Understanding Python objects doesn't have to be complicated. Here are the key points:

1. **Use `==` to compare values, `is` to check if it's the exact same object**
   - Like checking if two people have the same phone number (`==`) vs checking if they're the same person (`is`)

2. **Lists, dicts, and sets are like shared documents—everyone sees the changes**
   - Make a copy with `[:]` or `.copy()` if you need your own version

3. **Numbers, strings, and tuples are like sticky notes—changes make new ones**
   - You can't accidentally modify them, which makes them safer

**Common mistakes to avoid:**
- Don't use mutable objects (like lists) as default parameters
- Remember that `my_list2 = my_list1` doesn't create a copy
- When passing lists to functions, remember they can be modified

**When this really matters:**
- Building web applications with shared data
- Writing functions that shouldn't have side effects
- Debugging why your variables mysteriously changed
- Optimizing your code's memory usage

Understanding these concepts isn't about memorizing rules—it's about recognizing patterns. The next time you see unexpected behavior in your code, ask yourself: "Is this object mutable? Am I sharing a reference or making a copy?" More often than not, that's where your bug is hiding.

---

## References and Further Reading

- [Python Official Tutorial - Data Structures](https://docs.python.org/3/tutorial/datastructures.html)
- [Real Python - Pass by Reference in Python](https://realpython.com/python-pass-by-reference/)

---

**Links:**
- Blog Post: [Add your Medium/LinkedIn article URL here]
- LinkedIn Post: [Add your LinkedIn post URL here]
- GitHub Repository: [holbertonschool-higher_level_programming/python-everything_is_object](https://github.com/rpokman/holbertonschool-higher_level_programming/tree/main/python-everything_is_object)

---

*Written as part of the Holberton School curriculum - Learning Python the right way, from the ground up.*

