---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# ClientHub Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

This project is based on the AddressBook-Level3 project created by the [SE-EDU initiative](https://se-education.org).

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

**Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</box>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `ClientHubParser` returns back as a `Command` object.
* The `Command` objects contain examples of the appropriate format of the command, as well as the data passed in by the user
(e.g. `Person` object to be added in an `AddCommand` object)
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* handles predicate determination for keyword comparison of attributes in all `Person` objects
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<box type="info" seamless>

**Note:** An alternative model is given below. It has a `Set<Product>` in `Person`. This allows `AddressBook` to only
require one `Product` object per unique product, instead of each `Person` needing their own `Product` objects.<br>

<puml src="diagrams/BetterModelClassDiagram.puml" width="450" />

</box>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `ClientHubStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### Undo/redo feature

#### Implementation

The undo/redo mechanism is implemented in `ModelManager` with two stacks of deep-copied `AddressBook` states:

- `undoStack`: history of previous states
- `redoStack`: history of states that were undone and can be restored

Operations that modify stored data (e.g., `add`, `edit`, `delete`, `clear`) push a snapshot of the current `AddressBook` onto `undoStack` BEFORE applying the change, and CLEAR the `redoStack`.

- `setAddressBook(...)`: pushes a snapshot, clears `redoStack`, then replaces the state
- `addPerson(...)`: pushes a snapshot, clears `redoStack`, then adds
- `setPerson(...)`: pushes a snapshot, clears `redoStack`, then updates
- `deletePerson(...)`: pushes a snapshot, clears `redoStack`, then removes

Atomicity: For batch-like operations (e.g., delete-by-status), the command constructs an updated copy and calls `model.setAddressBook(updated)` ONCE so the entire batch is treated as a single undoable action.

`undo()`:
- Fails if `undoStack` is empty (`canUndo() == false`)
- Pushes the current state to `redoStack`
- Pops the top from `undoStack` and restores it via `addressBook.resetData(previous)`

`redo()`:
- Fails if `redoStack` is empty (`canRedo() == false`)
- Pushes the current state to `undoStack`
- Pops the top from `redoStack` and restores it via `addressBook.resetData(next)`

Command routing:
- `UndoCommand#execute` calls `model.undo()`; `RedoCommand#execute` calls `model.redo()`
- The parser recognizes `undo` and `redo` keywords and instantiates the respective commands

Persistence:
- After a command executes, `LogicManager` saves the current `AddressBook` via `storage.saveAddressBook(model.getAddressBook())`. This includes states after `undo` and `redo`, ensuring the reverted or restored state is persisted.

Diagrams:
- The existing undo/redo sequence diagrams remain applicable conceptually (command executes through Logic into Model, which reverts/restores the in-memory state and then is saved).

#### Design considerations

- Chosen approach: Snapshot the entire `AddressBook` for each modifying command
  - Pros: Simple and robust
  - Cons: Higher memory usage for large datasets
- Alternative: Command-specific inverse operations
  - Pros: Lower memory footprint
  - Cons: Higher complexity; more room for errors across many commands

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

* sales representatives
* tired of messy and complicated Excel sheets
* wants more efficient and seamless experience tracking potential clients
* has a need to manage a significant number of contacts
* prefer desktop apps over other types
* can type fast
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps

**Value proposition**:
Sales representatives will have more efficient control over tracking their client interaction and progress,
thereby improving their operational efficiency. This is especially so for users who are more comfortable with
CLIs compared to GUIs which can be complex and overwhelming

<br>

### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​                                                                      | I want to …​                                                                                  | So that I can…​                                                                                  |
|-----|------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| `* * *` | user                                                                         | delete a client                                                                               | remove clients that I am no longer servicing                                                     |
| `* * *` | user                                                                         | add clients                                                                                   | keep track of all potential clients                                                              |
| `* * *` | user                                                                         | view all my clients at one glance                                                             | review my full client list and update it if necessary                                            |
| `* * *` | new user who has not memorised the commands yet                              | be able to see the list of commands available and usage instructions                          | remember how to use the interface, and make use of ClientHub effectively                         |
| `* * *` | user with many clients to service                                            | save my client contact details                                                                | contact my clients                                                                               |
| `* * *` | meticulous sales representative who wants to manage clients thoroughly       | add the onboarding progress of my clients                                                     | keep track of the best leads to follow up                                                        |
| `* *` | sales representative whose clients need to change information                | edit my clients' information                                                                  | update their personal particulars and relevant information to be used for contacting them        |
| `* * ` | new user to ClientHub                                                        | see the client book populated with sample data                                                | see how the app is supposed to look like when I use it                                           |
| `* * ` | new user who wants to start using ClientHub                                  | remove all the current data                                                                   | get rid of all sample/test data used for exploring the app, and start adding real data           |
| `* * ` | user who accidentally deleted a client's information                         | go back to the previous history before the delete happened                                    | prevent any client information from being lost                                                   |
| `* *` | sales representative who is selling a range of products and services         | track my customers tagged with their respective deals                                         | remember which deal each client is interested in                                                 |
| `* *` | sales representative who is trying to get more sales quickly                 | list all my clients filtered by status                                                        | plan my daily cold calls more efficiently                                                        |
| `* *` | sales representative who struggles to remember names                         | find users with specific descriptions quickly                                                 | find someone in my contact even if I don't remember their name                                   |
| `* *` | sales representative who is constantly checking up on clients                | update the status of my clients                                                               | keep track of the clients that I have approached and clients that I have met                     |
| `* *` | sales representative who offers tailored products and services to clients    | add quick notes to a client record                                                            | personalise my follow ups and build better relationship                                          |
| `* *` | established sales representative with many clients                           | sort my client list by last contact or priority                                               | focus on the more urgent opportunities first                                                     |
| `* *` | sales representative with clients that can be easily stratified              | group related clients together                                                                | manage clients that are related to each other more easily                                        |
| `* *` | sales representative selling a range of products and services                | group clients by a certain product                                                            | view clients who have the same product easily, and update them all if there are changes          |
| `* *` | busy sales representative                                                    | colour code my client list                                                                    | visually see what category my clients fall under without having to open each client's details    |
| `* *` | sales representative with a wide range of clients                            | delete multiple clients at one go if they fit a certain criteria                              | easily remove unsuccessful or uncontactable clients                                              |
| `* *` | sales representative who likes to organise data properly                     | sort contact by name or tags or other information                                             | organise my client list easily and know who to approach specifically                             |
| `*` | busy sales representative                                                    | flag specific client contact as tasks to be done and put in details of the task               | easily see all the tasks I need to do urgently                                                   |
| `*` | busy sales representative                                                    | see all the contacts with pending tasks to be done                                            | easily see all the tasks without looking through the list                                        |
| `*` | competitive sales representative                                             | see potential clients that have approached my company but have yet to attach to any sales rep | reach out to more contacts and close more deals                                                  |
| `*` | sales representative who is joining ClientHub from other applications        | import my existing Excel/CSV contact list                                                     | transition smoothly into ClientHub without having to retype everything                           |
| `*` | sales representative who needs to share their client information with others | export my current client list                                                                 | transfer my client list to a different platform or device                                        |
| `*` | sales representative who needs to update others on their clients             | share my client details                                                                       | update my company or colleagues on my client's progress if necessary                             |
| `*` | sales representative who has many clients over a long time                   | view summary report of clients                                                                | see progress of clients over time                                                                |
| `*` | busy sales representative                                                    | schedule alerts to send messages to clients                                                   | send timely messages to update or contact clients                                                |
| `*` | forgetful sales representative with many clients                             | be able to see the contact details of my most frequently contacted clients                    | check up on their wellbeing and need for products/services, and build a strong rapport with them |
| `*` | sales representative                                                         | be able to put links to documents or sheets into contact details                              | keep track of more information about each client                                                 |

<br>

### Use cases

(For all use cases below, the **System** is the `ClientHub` and the **Actor** is the `user`, unless specified otherwise)

**Use case: UC01 - Add a client**

**MSS**
1.  User requests to add a client with all the required details.
2.  ClientHub adds the client to ClientHub.

    Use case ends.

**Extensions**
* 1a. The given client's details are of invalid format.

  * 1a1. ClientHub shows an error message.

    Use case ends.

* 1b. The given client's details are valid but have the same name and phone number as another client in ClientHub.

  * 1b1. ClientHub shows an error message.

    Use case ends.

**Use case: UC02 - Find clients**

**MSS**
1.  User requests to find clients with a specific attribute value.
2.  ClientHub shows a list of clients matching the search criteria.

    Use case ends.

**Extensions**
* 1a. No client found.

  * 1a1. ClientHub shows an error message.

    Use case ends.

* 1b. The given attribute values are of invalid format.

  * 1b1. ClientHub shows an error message.

    Use case ends.

**Use case: UC03 - Delete a client**

**MSS**
1.  User requests to delete a specific client by their index.
2.  ClientHub deletes the client.

    Use case ends.

**Extensions**
* 1a. The given index is invalid.

    * 1a1. ClientHub shows an error message.

      Use case ends.

**Use case: UC04 - Delete all clients with a specific status**

**MSS**
1.  User requests to delete all clients with a specific status.
2.  ClientHub deletes all matching clients.

    Use case ends.

**Extensions**
* 1a. No clients with the specified status exist.

    * 1a1. ClientHub shows an error message.

      Use case ends.

* 1b. The given status is invalid.

    * 1b1. ClientHub shows an error message.

      Use case ends.

**Use case: UC05 - Edit a client**

**MSS**
1.  User requests to edit a client by index with new details.
2.  ClientHub updates the client's details.

    Use case ends.

**Extensions**
* 1a. The given index is invalid.

    * 1a1. ClientHub shows an error message.

      Use case ends.

* 1b. The given details are of invalid format.

    * 1b1. ClientHub shows an error message.

      Use case ends.

**Use case: UC06 - List all clients**

**MSS**
1.  User requests to list all clients.
2.  ClientHub shows a list of all clients.

    Use case ends.

**Use case: UC07 - Clear all clients**

**MSS**
1.  User requests to clear all clients.
2.  ClientHub clears all clients from the list.

    Use case ends.

**Use case: UC08 - Undo previous command**

**MSS**
1.  User requests to undo the previous command.
2.  ClientHub reverts the previous action.

    Use case ends.

**Extensions**
* 1a. There are no previous commands to undo.

    * 1a1. ClientHub shows an error message.

      Use case ends.

**Use case: UC09 - Redo previous undone command**

**MSS**
1.  User requests to redo the previous undone command.
2.  ClientHub reapplies the previously undone action.

    Use case ends.

**Extensions**
* 1a. There are no undone commands to redo.

    * 1a1. ClientHub shows an error message.

      Use case ends.

**Use case: UC10 - View help**

**MSS**
1.  User requests help.
2.  ClientHub displays the help window with command instructions.

    Use case ends.

<br>

### Non-Functional Requirements

1.  Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2.  Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
3.  A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
4. The client list should be stored locally and be in a human editable text file.
5. ClientHub should be able to work without requiring an installer.
6. The GUI of ClientHub should not have any resolution-related inconveniences for the user, standard screen resolutions 1920x1080 and higher, and for screen scales 100% and 125%.
7. The GUI of ClientHub should be usable, even if user experience is not optimal, for resolutions 1280x720 and higher, and for screen scales 150%.
8. ClientHub should still be functional even if new commands, fields, or parameters are added in the future.
9. The commands should not take more than 5 seconds to run and carry out its response.
10. ClientHub should be usable by someone who cannot remember the commands, or is new to ClientHub.
11. ClientHub should not handle communication between clients and sales representatives.
12. Client list should be persistent and not change unless edited with a command or through text editor by user.
13. Users should not be able to see each others' client lists, they should be private and visible only to the current user.
14. The CLI should provide clear, actionable error messages for all invalid commands.

<br>

### Glossary
**Status**: Labels that are used to indicate the progress of communication or follow-up between the sales representative (user) and the client)
- *uncontacted* - The client has not been reached out to yet
- *inprogress* - Communication is ongoing with the client, but no outcome has been determined
- *successful* - A successful deal has been reached between the sales representative and the client
- *unsuccessful* - An attempt to deal with the client was not successful
--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<box type="info" seamless>

**Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</box>

### Launch and shutdown

1. Initial launch

   i. Download the jar file and copy into an empty folder

   ii. Double-click the jar file 
      -  Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.
          >    Alternatively, open a command terminal and `cd` into the folder.
          >    Then run `java -jar ClientHub.jar`
   
1. Saving window preferences

   i. Resize the window to an optimum size. Move the window to a different location. Close the window.

   ii. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

### Help

1. Viewing the help window

   i. Prerequisites: Application is running.

   ii. Test case: `help`<br>
      Expected: Help window opens. Status message indicates help opened. Command box remains usable.

   iii. Test case: `help something`<br>
      Expected: Help window opens. Status message indicates help opened. Command box remains usable.

### List

1. Listing all persons

   i. Prerequisites: Application contains at least one person.

   ii. Test case: `list`<br>
      Expected: All persons are shown. Status message indicates number of persons listed. Timestamp in the status bar is updated.

   iii. Test case: `list 123`<br>
      Expected: All persons are shown. Status message indicates number of persons listed. Timestamp in the status bar is updated.    

### Add

1. Adding a person with all required fields

   i. Prerequisites: None.

   ii. Test case: `add n/John Doe p/98765432 c/Apple e/john@apple.com a/1, Infinite Loop s/uncontacted`<br>
      Expected: New contact is added to the end of the list. Details shown in status message. Timestamp updated.

   iii. Test case: Duplicate add (same name and phone)<br>
      Expected: Error message indicating duplicate person. No changes to list. Status bar remains the same.

   iv. Test case: Missing required field (e.g. omit `n/`)<br>
      Expected: Error message indicating invalid command format. No changes to list.

### Edit

1. Editing fields of an existing person

   i. Prerequisites: List all persons using `list`. Multiple persons in the list.

   ii. Test case: `edit 1 n/Jane Doe e/jane@example.com`<br>
      Expected: First contact's name and email updated. Details of edited contact shown in status message. Timestamp updated.

   iii. Test case: `edit 0 n/Bob`<br>
      Expected: Error message indicating invalid index. No changes to list. Status bar remains the same.

   iv. Test case: `edit 1` (no fields)<br>
      Expected: Error message indicating at least one field must be provided. No changes to list.

   v. Test case: Edit resulting in duplicate (same name and phone as another contact)<br>
      Expected: Error message indicating duplicate person. No changes to list.

### Find

1. Finding persons by different fields

   i. Prerequisites: Have multiple persons with distinct names/companies/statuses/products.

   ii. Test case: `find n/John`<br>
      Expected: Shows persons with names containing "John" (case-insensitive). Status message indicates number of persons listed. Timestamp updated.

   iii. Test case: `find c/po`<br>
      Expected: Shows persons whose company contains "po" (case-insensitive substring).

   iv. Test case: `find s/successful`<br>
      Expected: Shows only persons with status exactly `successful`.

   v. Test case: `find` (no prefixes)
      
      Expected: Error message indicating at least one field is required. No changes to list.

### Clear

1. Clearing all entries

   i. Prerequisites: List contains at least one person.

   ii. Test case: `clear`<br>
      Expected: All contacts removed. Status message indicates clear success. Timestamp updated. A single `undo` restores all contacts.

### Undo

1. Undoing the last modifying command

   i. Prerequisites: Have executed at least one successful modifying command.

   ii. Test case: After `add ...`, run `undo`<br>
      Expected: The added contact is removed. Status message indicates undo success. Timestamp updated.

   iii. Test case: After `delete 1`, run `undo`<br>
      Expected: The deleted contact is restored. Status message indicates undo success. Timestamp updated.

   iv. Test case: No prior modifying command, run `undo`<br>
      Expected: Error message indicating nothing to undo. No changes to list or status bar.

### Redo

1. Redoing the most recently undone command

   i. Prerequisites: Perform a successful `undo`.

   ii. Test case: After undoing an add, run `redo`<br>
      Expected: The contact is added back. Status message indicates redo success. Timestamp updated.

   iii. Test case: After undoing a delete, run `redo`<br>
      Expected: The contact is deleted again. Status message indicates redo success. Timestamp updated.

   iv. Test case: No preceding `undo`, run `redo`<br>
      Expected: Error message indicating nothing to redo. No changes to list or status bar.

### Exit

1. Exiting the application

   i. Prerequisites: Application is running.

   ii. Test case: `exit`<br>
      Expected: Application closes. On next launch, window size/position and preferences are restored.

### Deleting a person

1. Deleting a person while all persons are being shown

   i. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

   ii. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   iii. Test case: `delete 0`<br>
      Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

   iv. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.


2. Deleting all persons with a specific status (atomic delete-by-status)

   i. Prerequisites: List all persons using `list`. Ensure at least one person with status `unsuccessful` (or another valid status) is present.

   ii. Test case: `delete unsuccessful`<br>
      Expected: All clients with status `unsuccessful` are deleted at once (single atomic change). Details of the deletion shown in the status message. Timestamp in the status bar is updated. A single `undo` should restore all deleted clients.

   iii. Test case: `delete something` (invalid status)<br>
      Expected: No person is deleted. Error message shown indicating invalid status. Status bar remains the same.

   iv. Test case: `delete successful` when there are no `successful` clients<br>
      Expected: No person is deleted. Error message shown indicating no matching persons. Status bar remains the same.

### Saving data

1. Dealing with missing/corrupted data files

   Handling data file issues

   i. Missing data file
   
   - Prerequisites: Application is closed. You know the data file path (`[JAR location]/data/addressbook.json`).
   - Test case: Delete or rename `addressbook.json`. Start the application.
     
     Expected: Application starts using sample data. No error dialog is shown. A new data file will be created on the next successful save.

   ii. Corrupted data file (invalid JSON or illegal values)
   
   - Prerequisites: Application is closed. You know the data file path.
   - Test case: Open `addressbook.json` and intentionally corrupt it (e.g. delete a closing brace, change field names, insert invalid values). Start the application.
     
     Expected: Application logs that the file could not be loaded and starts with an empty AddressBook. No crash; application remains usable.

   iii. Permission error on save
   
   - Prerequisites: Data file exists at `[JAR location]/data/addressbook.json`.
   - Test case (macOS/Linux): Make the data file or its folder read-only (e.g. `chmod a-w [JAR location]/data` or `chmod a-w addressbook.json`). Start the application and execute a modifying command (e.g. `add`, `delete`, `edit`, `clear`).
     
     Expected: Command executes in memory but saving fails. An error message is shown: "Could not save data to file ... due to insufficient permissions ..." (or a generic file I/O error). After restoring write permissions and executing another modifying command, saving succeeds.

