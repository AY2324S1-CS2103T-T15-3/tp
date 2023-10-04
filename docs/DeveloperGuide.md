---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# EduTrack Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

_{ list here sources of all reused/adapted ideas, code, documentation, and third-party libraries -- include links to the original source as well }_

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

### Architecture

<puml src="diagrams/ArchitectureDiagram.puml" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<puml src="diagrams/ArchitectureSequenceDiagram.puml" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<puml src="diagrams/ComponentManagers.puml" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<puml src="diagrams/LogicClassDiagram.puml" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

<puml src="diagrams/DeleteSequenceDiagram.puml" alt="Interactions Inside the Logic Component for the `delete 1` Command" />

<box type="info" seamless>

**Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.
</box>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<box type="info" seamless>

**Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<puml src="diagrams/BetterModelClassDiagram.puml" width="450" />

</box>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.addressbook.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedAddressBook#commit()` — Saves the current address book state in its history.
* `VersionedAddressBook#undo()` — Restores the previous address book state from its history.
* `VersionedAddressBook#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

<puml src="diagrams/UndoRedoState0.puml" alt="UndoRedoState0" />

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

<puml src="diagrams/UndoRedoState1.puml" alt="UndoRedoState1" />

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

<puml src="diagrams/UndoRedoState2.puml" alt="UndoRedoState2" />

<box type="info" seamless>

**Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</box>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

<puml src="diagrams/UndoRedoState3.puml" alt="UndoRedoState3" />


<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</box>

The following sequence diagram shows how the undo operation works:

<puml src="diagrams/UndoSequenceDiagram.puml" alt="UndoSequenceDiagram" />

<box type="info" seamless>

**Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</box>

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</box>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

<puml src="diagrams/UndoRedoState4.puml" alt="UndoRedoState4" />

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

<puml src="diagrams/UndoRedoState5.puml" alt="UndoRedoState5" />

The following activity diagram summarizes what happens when a user executes a new command:

<puml src="diagrams/CommitActivityDiagram.puml" width="250" />

#### Design considerations:

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire address book.
  * Pros: Easy to implement.
  * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
  * Pros: Will use less memory (e.g. for `delete`, just save the person being deleted).
  * Cons: We must ensure that the implementation of each individual command are correct.

_{more aspects and alternatives to be added}_

### \[Proposed\] Data archiving

_{Explain here how the data archiving feature will be implemented}_


--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**:

University tutor who:

* has a need to manage classes with a significant number of students
* prefer desktop apps over other types
* can type fast
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps

**Value proposition**: Each tutor has multiple groups of students where each group can be large, keeping track of students and records would become a hassle. Our product provides a centralised system that would help to organise all student records for easy and quick access.


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​  | I want to …​                              | So that I can…​                                                   |
|---------|----------|-------------------------------------------|-------------------------------------------------------------------|
| `* * *` | TA       | add a class                               | -                                                                 |
| `* * *` | TA       | remove a class                            | -                                                                 |
| `* * *` | TA       | view a class                              | view all students of a specific class                             |
| `* * *` | TA       | add a student                             | -                                                                 |
| `* * *` | TA       | remove a student                          | -                                                                 |
| `* * *` | TA       | save the data automatically               | ensure that my data is not lost when I forget to save             |
| `* *`   | TA       | update a class                            | keep the class information up to date                             |
| `* *`   | TA       | update a student                          | keep the student record up to date                                |
| `* *`   | TA       | take attendance in a lesson               | spend more time teaching                                          |
| `* *`   | TA       | monitor a lesson of a class               | keep track of administrative information for each lesson          |
| `* *`   | TA       | find a specific student by using a filter | quickly update their record such as attendance when they are late |
| `* *`   | new user | see the app populated with sample data    | easily see what the app looks like when it is in use              |

*{More to be added}*

### Use cases

(For all use cases below, the **System** is the `EduTrack` and the **Actor** is the `user`, unless specified otherwise)


**Use case: View all the information about a class**

**MSS**

1. User chooses a class out of all the classes they are teaching.
2. User requests to view all the information about that class.
3. EduTrack displays all the information about that class. (e.g. name, students, attendance, notes, etc)

**Use case: Add a class**

**MSS**

1.  User requests to add a class and specifies the class name
2.  EduTrack adds the class to the list of classes
3.  EduTrack shows the updated list of classes

**Extensions**

* 1a. Class name is not specified.

  * 1a1. EduTrack informs user that class name is empty.

      Use case ends.

* 1b. Class name already exists.

    * 1a1. EduTrack informs user that class already exists.

      Use case ends.

---

**Use case: Remove a class**

**MSS**

1.  User requests to view the list of classes
2.  EduTrack shows a list of classes
3.  User requests to delete a specific class in the list
4.  EduTrack requests for confirmation and waits for y/n response from user
5.  User confirms
6.  EduTrack deletes the class
7.  EduTrack informs the user that the class is successfully deleted


    Use case ends.

**Extensions**

* 1a. Class name is not specified.

    Use case ends.
* 2a. User did not specify the class.
  * 2a1. EduTrack informs user that class name is not specified in request.
  * 2a2. EduTrack terminates the request.

    Use case ends.

* 2b. EduTrack detects that the class does not exist.
  * 2b1. EduTrack informs user that the class does not exist.
  * 2b2. EduTrack terminates the request.

    Use case ends.

* 3a. The given class name is invalid.
    * 3a1. EduTrack shows an error message.

    Use case ends.

* 3b. No class name specified.
    * 3b1. EduTrack informs the user he should enter a class field

    Use case ends.

* 4a. User does not confirm, provides a `n` input
    * 4a1. EduTrack confirms that the cancellation is successful

    Use case ends.

---

**Use case: See the app populated with sample data**

**MSS**

1. User launches the app for the first time.
2. EduTrack populates the app with sample data for a class.
3. User accesses the sample data to see how the app works.

    Use case ends.

**Use case: Updating a Class note**

**MSS**

1.  User requests to view the list of classes
2.  EduTrack shows a list of classes
3.  User requests to update the Class notes of a particular class
4.  EduTrack overwrites the Class notes of that particular class
5.  EduTrack informs that user that the was successfully updated

**Extensions**
* 2a. The list is empty.

  Use case ends
* 3a. The given class name is invalid.
    * 3a1. EduTrack shows an error message.

      Use case ends.

* 3b. No class name specified.
    * 3b1. EduTrack informs the user he should enter a class field

      Use case ends.

* 3c. No note details was specified.
    * 3c1. EduTrack informs the user that no note was specified.

      Use case ends.

---

**Use case: Update a class**

**MSS**

1.  User requests to view the class to be updated
2.  EduTrack shows the class
3.  User requests to update a field in the class
4.  EduTrack updates the field of the class
5.  EduTrack shows the updated list of classes

    Use case ends.

**Extensions**
* 1a. Class name is not specified.

    * 1a1. EduTrack informs user that class name is empty.

      Use case ends.

* 1b. Class name already exists.

    * 1a1. EduTrack informs user that class already exists.

      Use case ends.

* 3a. Class field is not specified.

    * 3a1. EduTrack informs user that class field is empty.

      Use case ends.

* 3b. Class field does not exist.

    * 3b1. EduTrack informs user that class field does not exist.

      Use case ends.

* 3c. Updated information is not specified.

    * 3c1. EduTrack informs user that information to be updated is empty.

      Use case ends.

---

**Use case: Adding a lesson to a Class Schedule**

**MSS**

1.  User requests to view the list of classes
2.  EduTrack shows a list of classes
3.  User requests to add a lesson for a particular class
4.  EduTrack appends to the Class schedule of that particular class
5.  EduTrack informs the user that the lesson was added to the class schedule

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends
* 3a. The given class name is invalid.

    * 3a1. EduTrack shows an error message.

      Use case ends.

* 3b. No class name specified.
    * 3b1. EduTrack informs the user he should enter a class field

      Use case ends.
* 3c. No lesson details was specified.
    * 3c1. EduTrack informs the user that a lesson wasn't specified.

      Use case ends.
* 3d. Lesson details was of invalid format.
    * 3d1. EduTrack informs the user he should enter a lesson of the correct format

      Use case ends.

---

**Use case: Removing a lesson from a class schedule**

**MSS**

1.  User requests to view the list of classes
2.  EduTrack shows a list of classes
3.  User provides the lesson id to remove it from the class schedule
4.  EduTrack removes from the Class schedule of that particular class
5.  EduTrack informs the user that the lesson was removed from the class schedule

**Extensions**
* 2a. The list is empty.

  Use case ends
* 3a. The given class name is invalid.
    * 3a1. EduTrack shows an error message.

      Use case ends.

* 3b. No class name specified.
    * 3b1. EduTrack informs the user he should enter a class field

      Use case ends.
* 3c. No lesson id was specified.
    * 3c1. EduTrack informs the user that a lesson id wasn't specified.

      Use case ends.
* 3d. Lesson id is of invalid format or does not exist.
    * 3d1. EduTrack informs the user that the lesson does not exist.

      Use case ends.

---

**Use case: Take attendance in a lesson**

**MSS**

1.  User choose a class from the list of classes
2.  User requests to create a lesson of the class
3.  EduTrack creates a lesson monitor for the class
4.  User enters attendance of a student
5.  EduTrack updates the attendance field of the student
6.  User repeats step 3 for mutiple times
7.  User requests to end the lesson
8.  EduTrack ends the lesson and saves the data automatically

---

**Use case: Remove a student**

**MSS**

1. User chooses to remove a student from a class.
2. User requests to delete a specific student in a class.
3. EduTrack deletes the student from that class.

    Use case ends.

**Extensions**

* 1a. Class name is not specified.

    * 1a1. EduTrack informs user that class name is empty.

      Use case ends.

* 1b. Class name already exists.

    * 1a1. EduTrack informs user that class already exists.

      Use case ends.

* 4a. Student does not exist

    * 4a1. EduTrack informs user that student is not found.

      Use case ends.

* 2a. User did not specify the class.
    * 2a1. EduTrack informs user that class name is not specified in the request.
    * 2a2. EduTrack terminates the request.

      Use case ends.

* 2b. EduTrack detects that the student is not found in the class.
    * 2b1. EduTrack informs user that student is not found in class.
    * 2b2. EduTrack terminates the request.

      Use case ends.

* 2c. EduTrack detects that the class does not exist.
    * 2c1. EduTrack informs user that the class does not exist.
    * 2c2. EduTrack terminates the request.

      Use case ends.

---

**Use case: Update a class**

**MSS**
1. User requests to modify an existing student record in a class.
2. EduTrack responds with a list of categories for user to choose to modify.
3. User chooses a category to modify.
4. EduTrack requests for new input in the category.
5. User enters the new input for the category.
6. EduTrack updates and displays the new student record.

    Use case ends.

**Extensions**

* 1a. EduTrack detects that student or class does not exist.
    * 1a1. EduTrack informs the user that the student or the class does not exist.
    * 1a2. EduTrack terminates the request.

      Use case ends.

---

**Use case: Use auto-save feature**

**MSS**
1. User makes any form of request that alters the database.
2. EduTrack updates the database directly.

   Use case ends.

---

### Non-Functional Requirements

1.  **Environment requirement:**
   - Should work on any _mainstream OS_ as long as it has Java `11` or above installed.

2.  **Scalability:**
   - Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.

3.  **Usability:**
   - A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
   - The user interface should be intuitive and user-friendly to minimize the learning curve for TAs.

4.  **Performance**
   - The system should respond to user requests within a reasonable time frame (i.e. under 2 seconds).

*{More to be added}*

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, macOS
* **Database**: A storage for all data to be stored in.
--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<box type="info" seamless>

**Note:** These instructions only provide a starting point for testers to work on; testers are expected to do more *exploratory* testing.

</box>


### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

1. _{ more test cases …​ }_

### Deleting a person

1. Deleting a person while all persons are being shown

   1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

   1. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

1. _{ more test cases …​ }_

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

1. _{ more test cases …​ }_
