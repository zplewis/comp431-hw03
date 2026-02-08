# comp431-hw03

## Tasks

- read a forward-file (having the format of the file you created in HW2) and
convert the contents of the mail messages back to SMTP commands
  - each SMTP message generated will be written to standard output
- The program should both:
  - generate SMTP server messages
  - "listen" for the SMTP response messages that a real server would emit in
to those messages
- Since the program will not be communicating with a real SMTP server, the user
will have to simulate the server by typing in the appropriate SMTP response
message after each SMTP server message
- There is a QUIT command which follows the same grammar as the "DATA" command.
- the user will simulate providing the server response message
  - all of your program's inputs should be read from standard input (stdin)
- when parsing SMTP response messages, you should only parse for the response number and ignore
whatever text is also sent with the response message
- echo the entire SMTP response message you receive to standard error (stderr) and NOT standard input!

## Definitions

- **protocol** - a set of rules governing the exchange or transmission of data
between devices;
  - a set of conventions governing the treatment and especially the formatting
of data in an electronic communications system
  - a specification of the format/syntax for a set of message exchanges
- **grammar** - a set of rules that define how to form valid strings in a formal
language, like a programming language, specifying its syntax, not its meaning
  - a set of rules (or productions) about how to write statements that are valid for the
protocol
- **parser** - processes a string and determines whether or not a string confirms
to a grammar or not

## How email will work

- user agent runs a protocol, Simple Mail Transport Protocol (SMTP), to send a
message based on text to the local mail server (probably the server associated
with the user's email address)
- the local mail server contacts the destination mail server(s) via SMTP
- the destination mail server places the mail into the appropriate user's
mailbox
- recipient user retrieves mail via a mail access protocol
  - Post Office Protocol (POP)
  - Internet Mail Access Protocol (IMAP)

- This seems like there are three pieces:
  - the user agent
  - the local mail server
  - the destination mail server

- There is an RFC for SMTP for SMTP created by the Network Working Group

## Messages

### The SMTP "mail-from" message

- this is the message that is used by the user agent to send to the local mail
server
  - this message tells the mail server who is sending this message
- This "MAIL FROM" message has three components
  - a command/message name
    - string "MAIL FROM:"
  - a reverse path (a path to get back to the sender)
    - the email address of the sender
  - the terminator - something that you use to terminate the mesasge
    - the "carriage return-line feed" sequence
    - in Linux, this will be the newline character (does he mean just \n?)

- HW1: Write a Python program to recognize valid SMTP mail-from messages entered
via the keyboard
  - the teacher emphasizes that **HW1** is a text-parsing assignment, not a
networking assignment

### The SMTP "RCPT TO" message

## Grammar

The grammar specified for this assignment is written in what is called
**Backus-Naur Form (BNF)** with the two colons followed by an equal sign.

BNF is a language used to define languages. More specifically, it's a notation
that allows a specific way for writing down grammar rules, providing a clear and
organized method to describe how non-terminal (and terminal) symbols can be
combined.

Terminals are "indivisible units of a language", or a token.

```text
<response-code> ::= <resp-number> <whitespace> <arbitrary-text> <CRLF>
<resp-number> ::= "250" | "354" | "500" | "501" | "503"
<arbitrary-text> ::= any sequence of printable characters
<rcpt-to-cmd> ::= "RCPT" <whitespace> "TO:" <nullspace> <forward-path> <nullspace> <CRLF>
<data-cmd> ::= "DATA" <nullspace> <CRLF>
<mail-from-cmd> ::= "MAIL" <whitespace> "FROM:" <nullspace> <reverse-path> <nullspace> <CRLF>
<whitespace> ::= <SP> | <SP> <whitespace>
<SP> ::= " " | "\t" | /* the space or tab character */
<nullspace> ::= <null> | <whitespace>
<null> :== no character
<reverse-path> ::= <path>
<forward-path> ::= <path>
<path> ::= "<" <mailbox> ">"
<mailbox> ::= <local-part> "@" <domain>
<local-part> ::= <string>
<string> ::= <char> | <char> <string>
<char> ::= any one of the printable ASCII characters, but not any of <special> or <SP>
<domain> ::= <element> | <element> "." <domain>
<element> ::= <letter> | <name>
<name> ::= <letter> <let-dig-str>
<letter> ::= any one of the 52 alphabetic characters A through Z in upper case and a through z in
lower case
<let-dig-str> ::= <let-dig> | <let-dig> <let-dig-str>
<let-dig> ::= <letter> | <digit>
<digit> ::= "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
<CRLF> ::= "\n" /* the newline character */
<special> ::= "<" | ">" | "(" | ")" | "[" | "]" | "\" | "." | "," | ";" | ":" | "@" | """
```

- on the left size of `::=` of the rule (also called production) is called the **non-terminal**
- the vertical bar, `|`, is interpeted as a logical "or" operator and indicates
a choice between components that are mutually exclusive
  - I think "mutually exclusive" will be important to note
- another thing to note is that within this grammar, there is a hierarchy of
rules; for example, `<name>` depends on `<letter> <let-dig-str>`, so
those two rules must be defined first to properly check for `<name>`
- `<string>` looks like a group of one or more `<char>` matches in a row
- `<char>` specifically says **any one of the printable ASCII characters, so I
think that distinction should be watched out for as well. If there is a way to
determine whether a character is an ASCII character (like converting a character
to its numeric index within the ASCII chart), then it may be easier to weed out
characters that do not belong
- `<newline>` is **the newline character**, so I believe it is just `\n` and
does not include `\r` but this needs to be confirmed using the writeup for HW1
- comments will be important to check yourself to make sure that you are
implementing this exactly, no shortcuts or assumptions
- avoid trimming strings as that may cause the code to allow strings that should
fail tests to pass
- `<let-dig-str>` seems to follow the same convention as `<string>`, where the
match is equal to one or more of `<let-dig>` tokens

## Description

Write a Python program on Linux that reads lines of input from stdin
(standard in, the keyboard), echos the input to stdout (the screen), and
determines whether they are valid SMTP mail-from messages

- The video says that if the input line is a valid mail-from message, then print
to stdout `Sender ok` exactly
  - but what about **echos the input to stdout**? Is that actually true?
- If the input is NOT valid, print `ERROR -- name`, where `name` is the first
non-terminal in the grammar that is missing or incorrect
  - need more clarity on this
  - based on an announcement on Piazza, the HW1 writeup provides an explicit
list of which non-terminals can generate errors and explains why. This may
address this same question I was having
  - would I print the non-terminal with the angle brackets or just the text
  within the angle bracket (`<name>` versus `name`)
    - the answer would be print as `ERROR -- name`, do not include the angle
brackets

## Parsing

### HW1 Description Video

- `MAIL  FROM:<jeffay@cs.unc.edu>` - the error is too many spaces between `MAIL`
and `FROM`
  - this would be an error in the `<mail-from-cmd>`
- `MAIL FROM:<jeffay @cs.unc.edu>` - the error is a space after the
`<local-part>` and before the `@` sign
  - this would be an error in the `<mailbox>` rule
  - mail-from-cmd --> "MAIL" <SP> "FROM:" <reverse-path> --> "<" <mailbox>
--> <local-part> --> <let-dig-str> --> "@" (from <mailbox>) fails
  - you dig more into <let-dig-str>, but I skipped it here for brevity
- `MAIL FROM:<jeffay@cs.unc.edu >` - the error is in `<reverse-path>`
  - Within `<reverse-path>`, "<" passes, `<mailbox>` passes, but ">" fails
- things may behave differently from the video when you implement the full
grammar from HW1

### Recursive Descent Parser

- write a function for each non-terminal in the grammar
- the function's job is to either "recognize" that non-terminal or return an
unsuccessful indication
- each character should be processed individually from standard input

- Use a testfile called **testfile** containing all of the inputs to test

```bash
python parse.py < testfile
```

Think about how recursive functions work: you have a case for exiting the
function

- keep up with the next character to be processed
- to start, parse calls `mail_from_cmd`
-

## Assignment Submission

- name your file to be executed called `parse.py`
- create a subdirectory named HWx in your home directory
  - what is the server? `comp431-1sp26.cs.unc.edu`
  - server requires VPN if you are off-campus
- Before submitting any programming assignment, go back and read the assignment
description one more time!
- they are going to use the file modification date to determine when you turned
it in
  - make a copy if you have to make a change; certain editors change the
last modification date

To connect to the server:

```bash
ssh -l onyen comp431-1sp6.cs.unc.edu

# ssh onyen@comp431-1sp26.cs.unc.edu
```

- Python on this system is 3.12.3

## Notes to Self

- I'm not at the part where I actually code this yet, but I'm thinking that I
need a class for implementing the checks for each of these non-terminals in the grammar
- the homework mentions that "anything in square brackets (`[]`) is optional and
is not required to be present; however, I do not see any rules (also called productions) that
actually use square brackets, so I'm assuming that the grammar will be expanded
or changed in future assignments
