## Docstrings and Naming Conventions

The revisitation of ORM functionality ended up being a long (but necessary, I feel) diversion from the main project.


#### Naming Convention Practices
Always use Association Object. Always name Association Object as composite of the two objects' full names plus "assocation"

Object A : User
Object B : Message
Object C : UserMessageAssociation

Lookups always have Lookup at the end.
all relationship variables prefixed with "rel_"

# Actual database columns must reference database table names (foreign keys)
# relationships reference Python class objects
# When I link the objects in the Python code, I MUST do it on the RELATIONSHIP attributes,
# NOT the database columns!!!!!
# eg:

#   msg_type_global = MessageTypeLookup(type='Global')
#   msg_type_personnal = MessageTypeLookup(type='Personal')

#   msg1 = Message(rel_message_type=msg_type_global, text="This is message #1")
#   msg2 = Message(rel_message_type=msg_type_global, text="This is message #2")


DOCSTRING CONVENTIONS (VSCODE, useing '\t' causes display errors in Pylance).
Use of Docstring help to avoid needing extra long variable names (eg. rel_message_sender instead of rel_User_message_sender)
https://realpython.com/documenting-python-code/
Some display issues with Pylance. CHoice - write docstrings that looks good in VSCode (Markdown) or write docstrings that are compliant to the Python tooling (Sphinx, Google, etc). https://stackoverflow.com/questions/6046263/how-to-indent-a-few-lines-in-markdown-markup
https://github.com/sublimelsp/LSP-pyright/issues/42

Decision - follow Numpy style but make accommodations for VScode to make it usable (keep `\n and \t and code blocks`).
Did not find way to move docstrings out the class itself.

Next: TBD<br>
Previous: [Beef With ORMs](./09-orm-beef.md)<br>
