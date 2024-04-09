---
title: Adding and Updating CLI Commands
---
The Microblog api extends functionality from the Faker library to quickly enable owners to mock data in the sqlite db.

&nbsp;

## Usecase 1: Code Blocks

<SwmSnippet path="api/fake.py" line="9">

---

&nbsp;

```
faker = Faker()


@fake.cli.command()
```

---

</SwmSnippet>

Current supported switches are users and post. Swimm functionality enforces that when any lines encapsulated in the code block changes and alert is generated when navigating back to the documentation and if the change was pushed to the reppo the Swimm CI check fails.

<SwmSnippet path="api/fake.py" line="14">

---

This is the users function

```
def users(num):  # pragma: no cover
    """Create the given number of fake users."""
    users = []
    for i in range(num):
        user = User(username=faker.user_name(), email=faker.email(),
                    about_me=faker.sentence())
        db.session.add(user)
        users.append(user)

    # create some followers as well
    for user in users:
        num_followers = random.randint(0, 5)
        for i in range(num_followers):
            following = random.choice(users)
            if user != following:
                user.follow(following)

    db.session.commit()
    print(num, 'users added.')
```

---

</SwmSnippet>

<SwmSnippet path="/api/fake.py" line="38">

---

This the posts function

```
@click.argument('num', type=int)
def posts(num):  # pragma: no cover
    """Create the given number of fake posts, assigned to random users."""
    users = db.session.scalars(User.select()).all()
    for i in range(num):
        user = random.choice(users)
        post = Post(text=faker.paragraph(), author=user,
                    timestamp=faker.date_time_this_year())
        db.session.add(post)
    db.session.commit()
    print(num, 'posts added.')
```

---

</SwmSnippet>

&nbsp;

## Usecase 2: CLI Commands

These commands are run using sample CLI commands in the form

```bash
flask fake users <num>
flask fake posts <num>
```

I am unable to embed a smart token into a code block to link the snippet to underlying code which would allow sample CLI commands to inherit the revision aware functionality.

I am able to replicate the revision functionality below however it is more time consuming and less visually indicative of a copyable CLI command wrapped in a code block:

`flask fake `<SwmToken path="/api/fake.py" pos="14:2:2" line-data="def users(num):  # pragma: no cover">`users`</SwmToken>` <`<SwmToken path="/api/fake.py" pos="38:6:6" line-data="@click.argument(&#39;num&#39;, type=int)">`num`</SwmToken>`>`

`flask fake `<SwmToken path="/api/fake.py" pos="39:2:2" line-data="def posts(num):  # pragma: no cover">`posts`</SwmToken>` <`<SwmToken path="/api/fake.py" pos="38:6:6" line-data="@click.argument(&#39;num&#39;, type=int)">`num`</SwmToken>`>`

It's important to know that the smart tokens will only alert when the names they are referring to change. This means that if the <SwmToken path="/api/fake.py" pos="14:2:2" line-data="def users(num):  # pragma: no cover">`users`</SwmToken> function name changes or the <SwmToken path="/api/fake.py" pos="38:6:6" line-data="@click.argument(&#39;num&#39;, type=int)">`num`</SwmToken> parameter where to change the CI will flag, however if the logic of the function changes then no alert will flag. This should not be a major issue assuming we don't regularly leave the name of a function the same while radically changing the underlying logic (rather than refactoring).

&nbsp;

The core issue is that to my knowledge we do not store CLI commands within our codebase, nor do we have explicit integration tests for these commands.

&nbsp;

## Usecase 3: Scripting

Tracking changes that affects scripting code blocks (including jupyter notebooks) that do not live in the codebase suffers from the same issues as the CLI commands.

&nbsp;

```python
import random
from faker import Faker
from api.app import db
from api.models import User, Post

# This is how you fake 10 users with the same "About Me"

for i in range(10):
    user = User(username=faker.user_name(), email=faker.email(), about_me="I am the world's best manager")
    db.session.add(user)
```

Even if some of these individual lines existed in the codebase I would not want to use the code snippet functionality&nbsp;\
&nbsp;&nbsp;&nbsp;&nbsp;1.  Becuase the formatting would be inconsistent&nbsp;\
&nbsp;&nbsp;&nbsp;&nbsp;2. I do not want the link irelevant references in code snippets&nbsp;\
\
I could attempt to leverage smart_tokens as in the CLI example to jerry-rig a revisioned snippet however not only is that even more cumbersome than before given the length but there's also no guarantee that the coordination of the individual lines executes error free.

&nbsp;

### Alternatives for CLI and Scripting

In order to leverage Swimm while supporting usecases 2 & 3 we could implement testing requirements for CLI commands and scripting code blocks including functional and integration tests. All code snippets referenced in How-To documentation would require accompnying tests to ensure they continue to work. The Swimm documentation would then directly reference the underlying jupyter not ebooks, scripts and tests.&nbsp;

E.g

1. Create a jupyter notebook with all the snippets to be referenced in a doc. Place this file in a specific doc attachment directory (to be created)
2. Introduce github actions to run tests on files in this folder. [Nbval](https://nbval.readthedocs.io/en/latest/)
3. Reference these files inside Swimm documentation as Code snippetFunctional tests

&nbsp;

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBbWljcm9ibG9nLWFwaSUzQSUzQXJldHJlYWR1bnRhbWVk" repo-name="microblog-api"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
