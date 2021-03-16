+++
title = "Time tracking with Org Mode and sum time per tag"
author = ["Marco Pas"]
date = 2021-03-16T10:20:00+01:00
tags = ["emacs", "orgmode", "org", "timetracking"]
draft = false
+++

Tracking time using Org Mode is simple and easy. You can quickly create reports of the time spend on specific tasks. But how do you **aggregate time across tasks belonging to tags**?

This can be achieved by using a simple formula and the usage of an awesome Org package called [Org Aggregate](https://github.com/tbanel/orgaggregate).


## Input data {#input-data}

The data below is used for time tracking, note that individual items are **tagged**!

```text
- Take out the trash :private:
:LOGBOOK:
CLOCK: [2021-03-12 Fri 11:24]--[2021-03-12 Fri 11:30] =>  0:06
:END:
- Update document for client :client1:
:LOGBOOK:
CLOCK: [2021-03-12 Fri 12:45]--[2021-03-12 Fri 13:30] =>  0:45
:END:
- Create my awesome note for work :work:
:LOGBOOK:
CLOCK: [2021-03-13 Sat 11:24]--[2021-03-13 Sat 12:53] =>  1:29
:END:
- Fill in timesheet :work:
:LOGBOOK:
CLOCK: [2021-03-12 Fri 11:24]--[2021-03-12 Fri 11:40] =>  0:16
:END:
```


## Reporting {#reporting}

```text
#+BEGIN: clocktable :scope file :maxlevel 3 :tags t :match "work|client1" :header "#+TBLNAME: timetable\n"
#+TBLNAME: timetable
| Tags    | Headline                              | Time |      |      |     T |
|---------+---------------------------------------+------+------+------+-------|
|         | *Total time*                            | *2:30* |      |      |       |
|---------+---------------------------------------+------+------+------+-------|
|         | Report with filtered tags and sum...  | 2:30 |      |      |       |
|         | \_  Input data                        |      | 2:30 |      |       |
| client1 | \_    Update document for client      |      |      | 0:45 | 00:45 |
| work    | \_    Create my awesome note for work |      |      | 1:29 | 01:29 |
| work    | \_    Fill in timesheet               |      |      | 0:16 | 00:16 |
#+TBLFM: $6='(convert-org-clocktable-time-to-hhmm $5)::@1$6='(format "%s" "T")
#+END:
```

-   **:tags t** used to display the tags
-   **:match** "work|client'= used to filter the tags of interest
-   **:header** "#+TBLNAME: timetable\n"= used to name our table so we can process it later on using [Org Aggregate](https://github.com/tbanel/orgaggregate)
-   **#+TBLFM:** is using a function to correctly display time in **hh:mm** so we can use it later on to sum. **Note:** this is required as the package [Org Aggregate](https://github.com/tbanel/orgaggregate) that we are using to aggregate data is expecting the time in a **hh:mm** format

<!--listend-->

{{< highlight emacs-lisp "linenos=table, linenostart=1" >}}
(defun convert-org-clocktable-time-to-hhmm (time-string)
  "Converts a time string to HH:MM"
  (if (> (length time-string) 0)
      (progn
        (let* ((s (s-replace "*" "" time-string))
               (splits (split-string s ":"))
               (hours (car splits))
               (minutes (car (last splits)))
               )
          (if (= (length hours) 1)
              (format "0%s:%s" hours minutes)
            (format "%s:%s" hours minutes))))
    time-string))
{{< /highlight >}}

Use [Org Aggregate](https://github.com/tbanel/orgaggregate) to sum the times of the tags

```text
#+BEGIN: aggregate :table "timetable" :cols "Tags sum(T);U" :cond (not (equal Tags ""))
#+TBLNAME: timetable
| Tags    | sum(T);U |
|---------+----------|
| client1 |    00:45 |
| work    |    01:45 |
#+END:
```

-   **:cond** used to filter empty rows from the data input!
