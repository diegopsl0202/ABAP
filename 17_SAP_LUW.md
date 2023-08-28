<a name="top"></a>

# SAP LUW
- [SAP LUW](#sap-luw)
  - [Introduction](#introduction)
  - [Terms](#terms)
  - [SAP LUW Overview](#sap-luw-overview)
    - [Bundling Techniques](#bundling-techniques)
    - [Related ABAP Statements](#related-abap-statements)
  - [Concepts Related to the SAP LUW](#concepts-related-to-the-sap-luw)
  - [Notes on the SAP LUW in ABAP Cloud and RAP](#notes-on-the-sap-luw-in-abap-cloud-and-rap)
  - [More Information](#more-information)
  - [Executable Example](#executable-example)

## Introduction

⚠️ Most of the content of this cheat sheet is only relevant to [classic ABAP](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenclassic_abap_glosry.htm).

This cheat sheet provides a high-level introduction to the [SAP LUW](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abensap_luw_glosry.htm) concept that deals with data consistency with a focus on SAP LUW-related statements, supported by an executable example to check the syntax in action.

When you run an application, you typically change data in a [transaction](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abentransaction_glosry.htm), which may be temporarily stored in transactional buffers. 
This data may be temporarily inconsistent in the buffers, but it is important that the data be in a consistent state at the end of the transaction so that it can be saved to the database.

Consider the following example of transactional consistency: 
- A transaction consists of a money transfer from account A to account B, assuming the accounts are in the same bank and the data is stored in the same database.
- Such a transaction represents a logical unit. The transaction is successful when the money is debited from account A and credited to account B.
- This transaction may include other related tasks. Data may be loaded into a buffer, processed there, and become inconsistent during this time. It may also take a while for the whole process to be completed.
- However, at the end of the transaction, all data must be in a consistent state so that the database can be updated accordingly. Or, if errors occur during the transaction, it must be ensured that all changes can be reversed. It must not happen that money is credited to account B without also updating the totals of account A. In such a case, the previous consistent state must be restored.

> **💡 Note**<br>
> - This cheat sheet focuses on [classic ABAP](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenclassic_abap_glosry.htm). Hence, the links in this cheat sheet open topics in the ABAP Keyword Documentation for [Standard ABAP](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenstandard_abap_glosry.htm).
> - The SAP LUW concept is certainly relevant to ABAP Cloud, too. The [ABAP RESTful Application Programming Model (RAP)](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenarap_glosry.htm) is the transactional programming model for [ABAP Cloud](https://help.sap.com/doc/abapdocu_cp_index_htm/CLOUD/en-US/index.htm?file=abenabap_cloud_glosry.htm). It comes with a well-defined transactional model and follows the rules of the SAP LUW. Find out more in this [blog](https://blogs.sap.com/2022/12/05/the-sap-luw-in-abap-cloud/).

## Terms

The following terms are related to the concept of the SAP LUW and try to give you some context about it:

- [Transaction](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abentransaction_glosry.htm)
  - In a business context, a transaction describes a sequence of related and/or interdependent actions, such as retrieving or modifying data. 
  - The result of the transaction is a consistent state of data in the database.

- [Logical unit of work (LUW)](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenluw_glosry.htm)
  - Describes the time interval at which one consistent state of the database is transitioned to another consistent state.
  - Follows an all-or-nothing approach: It ends either with a single and final commit, which saves the changed data in the database, or with a rollback, which undoes all changes and restores the consistent state before the changes (for example, in the case of an error during the LUW). Either all data changes are committed, or none at all.
  - For an [Application Server ABAP (AS ABAP)](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenas_abap_glosry.htm), two types of LUWs come into play to achieve data consistency: Database LUW and SAP LUW (which is covered below).

- [Database LUW](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abendatabase_luw_glosry.htm)
  - Also called database transaction.
  - Is an SAP-independent mechanism for transactional consistency in the database.
  - Describes an indivisible sequence of database operations concluded by a [database commit](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abendatabase_commit_glosry.htm), that persists data to the database.
  - The [database system](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abendatabase_system_glosry.htm) either executes the database LUW completely or not at all. If an error is detected within a database LUW, a [database rollback](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abendatabase_rollback_glosry.htm) undoes all database changes made since the start of the database LUW.

- [Database commit](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abendatabase_commit_glosry.htm)
  - Marks the end of a database LUW in which changed data records are written to the database. 
  - An important question for developers is how and when database commits and rollbacks are triggered (especially implicitly). 
  - In AS ABAP, database commits can be triggered implicitly as well as by means of explicit requests.
  - Find more information [here](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abendb_commit.htm).

- Implicit database commits. Among others, implicit database commits are triggered by: 
  - Completing a [dialog step](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abendialog_step_glosry.htm) in the context of dynpros
    - A dialog step describes the state of a [user session](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenuser_session_glosry.htm) between a [user action](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenuser_action_glosry.htm) on the user interface of a dynpro and the sending of a new [screen layout](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenscreen_glosry.htm), i.e. it covers logic implemented in the [PAI](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenpai_glosry.htm) of the current dynpro and [PBO](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenpbo_glosry.htm) of the following dynpro 
    - When the next screen is displayed, the program waits for a user action and does not occupy a [work process](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenwork_process_glosry.htm) during this time.
    - The next free work process is assigned to the program in the next dialog step.
    - A work process change requires and implicitly triggers a database commit.
  - Calling a [function module](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenfunction_module_glosry.htm) in a [synchronous (sRFC)](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abensynchronous_rfc_glosry.htm) or [asynchronous remote function call (aRFC)](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenasynchronous_rfc_glosry.htm)
    - This is when the current work process passes control to another work process or system. Exception: [Updates](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenupdate_glosry.htm). 
  - HTTP/HTTPS/SMTP communication executed using the [Internet Communication Framework (ICF)](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenicf_glosry.htm)
  - [`WAIT`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abapwait_up_to.htm) statements that interrupt the current work process 
  - Sending messages ([error messages](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenerror_message_glosry.htm), [information message](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abeninformation_message_glosry.htm), and [warning](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenwarning_glosry.htm))

- Explicit database commits. For example, database commits can be triggered explicitly in ABAP programs in the following ways:
  - Using the relevant database-specific [Native SQL](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abennative_sql_glosry.htm) statement
  - Using the ABAP SQL statement [`COMMIT CONNECTION`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abapcommit_rollback_connection.htm)
  - Calling the function module `DB_COMMIT`, which encapsulates the corresponding Native SQL statement. 
  - Using the ABAP SQL statement [`COMMIT WORK`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abapcommit.htm). Note that the statement is particularly relevant to the SAP LUW as shown below. It also ends the SAP LUW. 
  
- [Database rollback](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abendatabase_rollback_glosry.htm)
  - Like a database commit, ...
    - a database rollback marks the end of a database LUW. Here, all modifying database operations are undone until the beginning of the LUW. 
    - are triggered implicitly, as well as by explicit requests.
  - They are implicitly triggered, for example, by a [runtime error](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenruntime_error_glosry.htm) or a [termination message](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abentermination_message_glosry.htm) (message of type `A`).
  - For example, explicit rollbacks are triggered by: 
    - Using the relevant database-specific Native SQL statement
    - Using the ABAP SQL statement [`ROLLBACK CONNECTION`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abapcommit_rollback_connection.htm) 
    - Calling the function module `DB_ROLLBACK`, which encapsulates the corresponding Native SQL statement. 
    - Using the ABAP SQL statement [`ROLLBACK WORK`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abaprollback.htm). Note that the statement is particularly relevant to the SAP LUW as shown below. It also ends the SAP LUW. 

- [Work process](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenwork_process_glosry.htm)
  - As a component of an AS ABAP AS instance, work processes execute ABAP applications. Each ABAP program that is currently active requires a work process.
  - AS ABAP uses its own work processes to log on to the database system. Different types of work processes are available for applications, including dialog, enqueue, background, spool, and update work processes.
  - As mentioned earlier, in dialog processing, a work process is assigned to an ABAP program for the duration of a dialog step. An application program can be divided into several program sections and, in the case of dynpros, into several dialog steps that are processed sequentially by different work processes.
  - For example, in the context of a dynpro, when a dialog step that is waiting for user interaction completes, the work process is released (for another workload) and a new work process is assigned. The current workload is persisted and resources are released. This approach requires an (implicit) database commit that ends the database LUW. 
  - A work process can execute only a single database LUW. It cannot interfere with the database LUWs of other work processes.


<p align="right">(<a href="#top">⬆️ back to top</a>)


## SAP LUW Overview

For an [SAP LUW](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abensap_luw_glosry.htm), the following aspects come into play: 

- Usually, an SAP LUW is started by opening a new [internal session](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abeninternal_session_glosry.htm). The execution of programming units can be distributed among several work processes.
- Database commits persist data in the database (especially implicit database commits when work processes are switched).
- The all-or-nothing rule applies: All database changes that occur during a transaction constitute a logical unit of work. They must be committed together, or rolled back together in the event of an error.
- This means that all database changes must be deferred and made in a final database commit (that is, in a single database LUW) at the end of a transaction - not in between.
- To ensure data consistency, all database changes are bundled, for example, in temporary tables or transactional buffers, and then, at the end of the transaction, all changes are written to the database together in a single work process, that is, in a single database LUW with a single database commit. This can be done directly using ABAP SQL statements at this stage, or using bundling techniques - special ABAP programming techniques.
  
Using the above bank transfer as an example: 
- At the end of the transaction, the new totals of both accounts are updated (money is debited from account A and credited to B).
- You cannot debit account A in one work process and then credit account B in a separate work process. When the work process changes, new totals would be available in one account, but not in the other. Consider the consequences if an error occurs and the new totals for account B cannot be updated, and so on. You would no longer be able to easily roll back the changes.
- Consider prematurely updating the database and notifying the users or processing the data while the logical unit has not been successfully completed.

<p align="right">(<a href="#top">⬆️ back to top</a>)

### Bundling Techniques
The following bundling techniques are available for classic ABAP. This means that programming units are registered in different work processes, but are executed by a single work process. All database changes are put into one database LUW, and all changes are committed in one final database commit.

**Using [update function modules](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenupdate_function_module_glosry.htm)**
- Are specially marked, i.e. the *update module* property is marked  
- Can be given specific attributes to determine the priority with which they are processed in the update work process
- Usually contain database modification operations/statements
- [CALL FUNCTION ... IN UPDATE TASK](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abapcall_function_update.htm) statements are used to register the update function modules for later execution; the actual execution is triggered by a `COMMIT WORK` statement

- Example of a simple update function module that has an importing parameter (a structure that is used to modify a database table)
  ```abap
  FUNCTION zsome_update_fu_mod
    IMPORTING
      VALUE(values) TYPE some_dbtab.

    MODIFY some_dbtab FROM values.

  ENDFUNCTION.
  ```

- Depending on your use case, you can run the update work process in several ways:
  - [Synchronous update](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abensynchronous_update_glosry.htm): The calling program waits until the update work process has finished. The `COMMIT WORK` statement with the `AND WAIT` addition triggers a synchronous update in a separate update work process.
  - [Asynchronous update](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenasynchronous_update_glosry.htm): The calling program does not wait for the update work process to finish. The `COMMIT WORK` statement triggers an asynchronous update in a separate update work process.
  - [Local update](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenlocal_update_glosry.htm): The update is performed immediately in the current work process in a separate internal session and not in a separate update work process, so that a synchronous update is always performed (regardless of whether `COMMIT WORK` is used with `AND WAIT` or not). By default, the local update is deactivated at the start of each SAP LUW. If required, you can activate local update for an SAP LUW using the [`SET UPDATE TASK LOCAL`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abapset_update_task_local.htm) statement before registering the update function modules.


    ```abap
    "Synchronous update

    DATA(st_a) = VALUE some_type( ... ).

    ... 

    "Registering the update function module specified in uppercase letters
    CALL FUNCTION 'ZSOME_UPDATE_FU_MOD' IN UPDATE TASK 
        EXPORTING values = st_a.

    ...

    "Triggering the synchronous update
    COMMIT WORK AND WAIT.                              

    ***********************************************************************
    "Asynchronous update

    DATA(st_b) = VALUE some_type( ... ).

    ... 

    CALL FUNCTION 'ZSOME_UPDATE_FU_MOD' IN UPDATE TASK 
        EXPORTING values = st_b.

    ...

    "Triggering the asynchronous update
    COMMIT WORK.

    ***********************************************************************
    "Local update

    "Before update function modules are registered.
    SET UPDATE TASK LOCAL.

    DATA(st_c) = VALUE some_type( ... ).

    ... 

    CALL FUNCTION 'ZSOME_UPDATE_FU_MOD' IN UPDATE TASK 
        EXPORTING values = st_c.

    ...

    "The update will be synchronous.
    COMMIT WORK.
    ```

> **💡 Note**<br>
> If a runtime error occurs during the update, the update work process executes a database rollback, and notifies the user whose program created the entries.

**Using [remote-enabled function modules](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenremote_enabled_fm_glosry.htm)**
- Also in this case, the bundling is done through function modules. 
- They are also specially marked as remote-enabled function modules.
- For example, you can use register them for later asynchronous execution in the background and through the [RFC interface](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenrfc_interface_glosry.htm) ([background Remote Function Call (bgRFC)](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenbg_remote_function_glosry.htm)). With this technology, you can make calls in the same or different ABAP systems. 
- More information:
  - [SAP Help Portal documentation about RFC](https://help.sap.com/docs/ABAP_PLATFORM_NEW/753088fc00704d0a80e7fbd6803c8adb/4888068AD9134076E10000000A42189D)
  - [`CALL FUNCTION ... IN BACKGROUND UNIT`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abapcall_function_background_unit.htm)

**Using [subroutines](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abensubroutine_glosry.htm)**
- Subroutines that are no longer recommended for use can be registered for later execution.

- They are registered with the [`PERFORM ... ON COMMIT`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abapperform_on_commit.htm) statement. These subroutines are executed when a `COMMIT WORK` statement is called.
- An addition is available to control the order of execution.
- Similarly, a subroutine can be registered "on rollback" with [`PERFORM ... ON ROLLBACK`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abapperform_on_commit.htm). These subroutines are executed when a `ROLLBACK WORK` statement is called.
- When executed: 
  - In the current work process, before update function modules.
  - When they are registered in an update function module with `ON COMMIT`, they are executed at the end of the update. This happens in the update work process for non-local updates, and in the current work process for local updates.

<p align="right">(<a href="#top">⬆️ back to top</a>)

### Related ABAP Statements

An SAP LUW is usually started by opening a new internal session. 
The statements to end an SAP LUW have already been mentioned above: [`COMMIT WORK [AND WAIT]`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abapcommit.htm) and [`ROLLBACK WORK`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abaprollback.htm).

`COMMIT WORK [AND WAIT]`
- Closes the current SAP LUW and opens a new one.
- Commits all change requests in the current SAP LUW. 
- Among other things, this statement triggers ...
  - the processing of all registered update function modules. 
  - the update work process and the local updates in the current work process. 
  - a database commit for all currently open [database connections](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abendatabase_connection_glosry.htm), which also terminates the current database LUW. 
- Note that `COMMIT WORK` triggers an asynchronous, `COMMIT WORK AND WAIT` a synchronous update.

`ROLLBACK WORK`
- Similar to `COMMIT WORK` statements, this statement closes the current SAP LUW and opens a new one.
- Among other things, this statement ...
  - causes all changes within a SAP LUW to be undone, that is, all previous registrations for the current SAP LUW are removed.
  - triggers a database rollback on all currently open database connections, which also terminates the current database LUW. 

> **💡 Note**<br>
> Notes on database connections:
> - The [database interface](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abendatabase_interface_glosry.htm) uses the [standard connection](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenstandard_db_connection_glosry.htm) of the current work process to access the [standard database](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenstandard_db_glosry.htm) by default.
> - Optionally, database accesses can also be made by using [secondary connections](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abensecondary_db_connection_glosry.htm) to [secondary databases](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abensecondary_db_glosry.htm) or by using [service connections](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenservice_connection_glosry.htm) to the standard database. The secondary connections are usually used by technical components. For example, they are used for caches, traces, logs, and so on.
> - The implicit database rollback is performed on all database connections that are currently open.
> - Within the SAP LUW, database changes and commits are allowed on service connections or through secondary database connections.


<p align="right">(<a href="#top">⬆️ back to top</a>)

## Concepts Related to the SAP LUW
The following concepts are related to the SAP LUW to ensure transactional consistency. They are not discussed in detail here. For more information, see the links.

**Authorization concept**
- In an SAP system, you need to protect data from unauthorized access by making sure that only those [authorized](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenauthorization_glosry.htm) to access it can see and modify it.
- Authorization to access data can be set. Before a user can perform certain operations in your application, you need to implement authorization checks.
- Since ABAP SQL statements do not trigger any [authorization checks](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenauthorization_check_glosry.htm) in the database system, this is even more important. Database tables may be accessed without restriction using these statements. Conversely, not all users in a system are authorized to access all data available to ABAP SQL statements.
- Thus, it is up to the programmer to ensure that each user who can call the program is authorized to access the data it handles.
- More information: 
  - [Authorizations](https://help.sap.com/doc/abapdocu_cp_index_htm/CLOUD/en-US/index.htm?file=abenbc_authority_check.htm)
  - [`AUTHORITY-CHECK`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abapauthority-check.htm)


**Lock concept**
- The database system automatically sets [database locks](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abendatabase_lock_glosry.htm) when ABAP SQL statements are called to modify database table entries.
- These locks are implemented by automatically setting a lock flag, which can only be set for existing database table entries.
- After a database commit, these flags are removed.
- As a result, database locks are not available for more than one database LUW, which must be considered in the context of an SAP LUW, since multiple database LUWs may be involved. Therefore, the lock flags that are set in a transaction are not sufficient. For the duration of an entire SAP LUW, a lock on database entries must remain set.
- This is where the SAP lock concept comes into play, which is independent of the automatic database locks. It is based on [lock objects](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenlock_object_glosry.htm).
- Lock objects ...
  - are [repository objects](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenrepository_object_glosry.htm) that are defined in the [ABAP Dictionary](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenabap_dictionary_glosry.htm).
  - specify the database tables in which records are to be locked with a lock request. 
  - contain the key fields on which a lock is to be set. 
- When a lock object is created, two [lock function modules](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenlock_function_module_glosry.htm) (`ENQUEUE_...` and `DEQUEUE_...`) are automatically generated. They are executed in a special enqueue work process. When a record is locked during a transaction (by the enqueue function module), a central lock table is filled with the table name and key field information. Unlike database locks, a locked entry in a lock object does not necessarily have to exist in a database table. Also, the locking must be done proactively, i. e. there is no automatic locking. You must make sure that the application implementation checks the lock entries. 
- At the end of an SAP LUW, all locks should be released, either automatically during the database update or explicitly when you call the corresponding dequeue function module.
- More information: [SAP Locks](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abensap_lock.htm)  

> **💡 Note**<br>
> RAP comes with its own implementation features to cover these concepts. See the topics [Authorization Control](https://help.sap.com/docs/SAP_S4HANA_CLOUD/e5522a8a7b174979913c99268bc03f1a/375a8124b22948688ac1c55297868d06.html) and [Concurrency Control](https://help.sap.com/docs/SAP_S4HANA_CLOUD/e5522a8a7b174979913c99268bc03f1a/d315c13677d94a6891beb3418e3e02ed.html) in the *Development guide for the ABAP RESTful Application Programming Model*.

<p align="right">(<a href="#top">⬆️ back to top</a>)

## Notes on the SAP LUW in ABAP Cloud and RAP

A limited set of ABAP language features is available in ABAP Cloud ([restricted ABAP language version](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenrestricted_version_glosry.htm)). 
The limitations include the fact that the above bundling techniques are not available. Note that the local update is enabled by default in ABAP Cloud. Find more information in this [blog](https://blogs.sap.com/2022/12/05/the-sap-luw-in-abap-cloud/).

In fact, RAP is the transactional programming model for ABAP Cloud.
And RAP comes with a well-defined transactional model and follows the rules of the SAP LUW. At the end of an SAP LUW in RAP, database modification operations should be performed in a final step in the [RAP late save phase](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenlate_rap_save_phase_glosry.htm) by persisting the consistent data in the [RAP transactional buffer](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abentransactional_buffer_glosry.htm) to the database.

There are RAP-specific, [ABAP EML](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abenabap_eml_glosry.htm) statements for commit and rollback: 
- [`COMMIT ENTITIES`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abapcommit_entities.htm) implicitly triggers `COMMIT WORK`. Furthermore, `COMMIT ENTITIES` provides RAP-specific functionality with various additions. These EML statements implicitly enforce local updates with `COMMIT WORK`, or `COMMIT WORK AND WAIT` if the local update fails. Therefore, the update is either a local update or a synchronous update, but never an asynchronous update. 
- [`ROLLBACK ENTITIES`](https://help.sap.com/doc/abapdocu_latest_index_htm/latest/en-US/index.htm?file=abaprollback_entities.htm): Resets all changes of the current transaction and clears the transactional buffer. The statement triggers `ROLLBACK WORK`. 

## More Information
- [The RAP Transactional Model and the SAP LUW](https://help.sap.com/docs/SAP_S4HANA_CLOUD/e5522a8a7b174979913c99268bc03f1a/ccda1094b0f845e28b88f9f50a68dfc4.html) (Development guide for the ABAP RESTful Application Programming Model)
- [The SAP LUW in ABAP Cloud](https://blogs.sap.com/2022/12/05/the-sap-luw-in-abap-cloud/) (blog)
- [Cheat sheet about ABAP EML](08_EML_ABAP_for_RAP.md)

<p align="right">(<a href="#top">⬆️ back to top</a>)

## Executable Example
Coming soon ...