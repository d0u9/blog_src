---
title: 'Applying the Linus Torvalds "Good Taste" Coding Requirement'
categories:
  - Science & Technology
  - Linux
tags:
  - Linux
  - Linux Kernel
  - Linus Torvalds
abbrlink: 315230980
date: 2017-01-18 08:31:00
---

**NOTE**: This post is cited from [this page](https://medium.com/@bartobri/applying-the-linus-tarvolds-good-taste-coding-requirement-99749f37684a#.rvr5414nj).

In a recent interview with Linus Torvalds, the creator of Linux, at approximately 14:20 in the interview, he made a quick point about coding with "good taste". Good taste? The interviewer prodded him for details and Linus came prepared with illustrations.

<!-- more -->

He presented a code snippet. But this wasn’t “good taste” code. This snippet was an example of poor taste in order to provide some initial contrast.

```C
remove_list_entry(entry)
{
        prev = NULL;
        walk = head;

        // Walk the list

        while (walk != entry) {
                prev = walk;
                walk = walk->next;
        }

        // Remove the entry by updating the
        // head or the previous entry

        if (!prev)
                head = entry->next;
        else
                prev->next = entry->next;
}
```

It’s a function, written in C, that removes an object from a linked list. It contains 10 lines of code.

He called attention to the if-statement at the bottom. It was this if-statement that he criticized.

I paused the video and studied the slide. I had recently written code very similar. Linus was effectively saying I had poor taste. I swallowed my pride and continued the video.

Linus explained to the audience, as I already knew, that when removing an object from a linked list, there are two cases to consider. If the object is at the start of the list there is a different process for its removal than if it is in the middle of the list. And this is the reason for the "poor taste" if-statement.

But if he admits it is necessary, then why is it so bad?

Next he revealed a second slide to the audience. This was his example of the same function, but written with "good taste".

```C
remove_list_entry(entry)
{
        // The "indirect" pointer points to the
        // *address* of the thing  we'll update

        indirect = &head;

        // Walk the list, looking for the thing that
        // points to the entry we want to remove

        while ((*indirect) != entry)
                indirect = &(*indirect)->next;

        // .. and just remove it

        *indirect = entry->next;
}
```

The original 10 lines of code had now been reduced to 4.

But it wasn’t the line count that mattered. It was that if-statement. It’s gone. No longer needed. The code has been refactored so that, regardless of the object’s position in the list, the same process is applied to remove it.

Linus explained the new code, the elimination of the edge case, and that was it. The interview then moved on to the next topic.

---

### ¶ The end
