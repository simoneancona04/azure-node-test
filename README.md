# Server configuration

## Table of Contents
1. [Get methods](#get-methods)
2. [Post methods](#post-methods)
3. [queries.json file (for server dev)](#queries-json)
4. [Queries](#queries)
5. [Actions](#actions)
6. [Errors](#errors)


## Get methods
- `\` web page request (single page application)

## Post methods
- `\` database queries
- `\var\` change server variables
- `\action\` make something

### Post `\` JSON format (client request)
The post request _must_ contains a JSON string that follows this format:
```json
{
    "name": "queryName",
    "argv": [

    ],
    "page": 0
}
```
where `name` is the name of the query, `argv` is the list of arguments, and `page` is the page number (starting from 0), i.e the range from the starting row, defined by the operation `page` * `maxRowVisualization`, to ending row, defined by the operation (`page` + 1) * `maxRowVisualization` - 1. 
> If page is set to -1, the row limitation will be ignored.

### Post `\` JSON format (client response)
The post respone strictly depends on the query executed. The data is sent according to this scheme
```json
[
    { "column0-name": "value", "column1-name": "value", "columnX-name": "value"},
    { "column0-name": "value", "column1-name": "value", "columnX-name": "value"},

    { "column0-name": "value", "column1-name": "value", "columnX-name": "value"},
]    
```
If the query was an add query, the response json follows this scheme
```json
{
    "affectedRows": 1,
    "fieldCount": 0,
    "info": "string",
    "insertId": 0,
    "serverStatus": 2,
    "warningStatus": 0,
}
```

### Post `\var\` JSON format (client request)
This post is used to change server variables, the json should follows this scheme
```json
{
    "name": "varName",
    "method": "set | add",
    "value": 0
}
```

### Post `\action\` JSON format (client request)
This post is used to perform different type of operations, the json follows this scheme
```json
{
    "name": "actionName",
}
```
The response does not follow a scheme.

<div id='queries-json'></div>

## queries.json file (for server developer)
```json
{
    "name": "string",
    "query": "string",
    "argc": 0
}
``` 
where `name` is the name of the action, `query` is the query to make to the server and `argc` is the number of accepted arguments.

## Queries

### The name of the query
The name of the query starts with the name of the table where to get the data.
The second part of the name is a condition.  
For example the query:  
`teacher-name` select, from the teacher table, all teachers with that name. 

### Teacher table
- `teacher` select all from teacher table. `argc` = 0  
- `teacher-no-id` select all except id from teacher table. `argc` = 0
- `teacher-name` select teachers with names starting with the given string (NameInitial).  
    - `argc` = 1  
    - `argv` = (NameInital: string)
- `teacher-name-or-surname` select teachers with names or surnames starting with the given string (NameInitial).  
    - `argc` = 1  
    - `argv` = (NameInital: string)
- `teacher-id` select the teacher with that id.  
    - `argc` = 1
    - `argv` = (TeacherID: int)
- `teacher-add` add a teacher  
    - `argc` = 2  
    - `argv` = (Name: string, Surname: string)

### Absence table
- `absence` select all from absence table. `argc` = 0
- `absence-no-id` select all  except id from absence table. `argc` = 0

### Course table
- `course` select all from course table. `argc` = 0
- `course-teacher-name` select all the courses in which a particular teacher (referred by the name) teaches.  
    - `argc` = 1  
    - `argv` = (Name: string)
- `course-teacher-name-surname` select all the courses in which a particular teacher (referred by the name and surname) teaches.
    - `argc` = 2
    - `argv` = (Name: string, Surname: string)
- `course-teacher-id` select all the courses in which a particular teacher (referred by the ID) teachs.
    - `argc` = 1
    - `argv` = (TeacherID: int)
- `course-location-id` select all the courses in a particular location.
    - `argc` = 1
    - `argv` = (LocationID: int)
- `course-join-location` join between course table and location table. `argc` = 0
- `course-join-location-serach` like `course-join-location` but select the course with a specific year or section.
    - `argc` = 1
    - `argv` = (SearchingString: string)
- `course-add` add a new course
    - `argc` = 3
    - `argv` = (Year: int, Section: string, LocationID: int)

### Subject table
- `subject` select all from subject table. `argc` = 0
- `subject-add` add a subject.  
    - `argc` = 1  
    - `argv` = (SubjectName: string)

### Substitution table
- `substitution` select all from substitution table. `argc` = 0
- `substitution-no-id` select all except id from substitution table. `argc` = 0

### Teaching table
- `teaching` select all from teaching table. `argc` = 0
- `teaching-no-id` select all except id from teaching table. `argc` = 0
- `teaching-course` select a teaching with a specific course.  
    - `argc` = 2  
    - `argv` = (CourseYear: int, CourseSection: string)
- `teaching-add` add a teaching  
    - `argc` = 6  
    - `argv` = (Day: int, Hour: int, TeacherID: int, SubjectName: string, CourseYear: int, CourseSection: string)
- `teaching-course-teacher` select the course and the teacher (TeacherID) from the teaching table. `argc` = 0
- `teaching-join-teacher` similar to `teaching-course-teacher` but the teacher is referred by the name and surname. `argc` = 0
> Remember that the Day in the teaching table is an integer that rappresent the day of the week, where 1 is Monday and 7 is Sunday.

### Location
- `location` select all from location. `argc` = 0
- `location-add` add a nwe location.
    - `argc` = 3
    - `argv` = (Name: string, EntryTime: string[`"hh:mm:ss"`], ModuleDuration: string[`"hh:mm:ss"`])

## Actions

### getTeacherCourses
This action sends as a response a json that contains all the teachers with their respective courses. The json response should look like this:
```json
[
    {"TeacherID": 1, "Name": "John", "Surname": "Smith", "SubstitutionDone": 0, "Courses": [
        {"Year": 1, "Section": "A", "LocationID": 2}
    ]}
]
```

## Errors

### Json format
```json
{
    "error": "string | JSON",
    "type": "string"
}
```
where `error` could be a string (error message) or an object that contains other error infromation, and `type` is the type of the error

### DB_CONNECTION_ERROR
it occurs when the connection cannot be established

### DB_END_CONNECTION_ERROR
it occurs when the connection cannot be closed

### DB_QUERY_ERROR
it can occur when a query is sent

### QUERY_NOT_FOUND_ERROR
it occurs when the specified query name is not in the above list

### QUERY_ARGUMENTS_ERROR
it occurs when the arguments passed are incorrect

### ACTION_NOT_FOUND_ERROR
same as `QUERY_ARGUMENTS_ERROR` but on action

## Database Configuration
![database scheme](./doc/scheme.PNG)

### Teacher table

| _TeacherID_   | Name         | Surname        | SubstitutionDone |
|---------------|--------------|----------------|------------------|
| int           | varchar(255) | varchar(255)   | int              |

### Absence table

| _AbsenceID_   | Day   | Hour  | TeacherID | SubstitutionID |
|---------------|-------|-------|-----------|----------------|
| int           | int   | int   | int       | int            |

### Course table

| _Section_   | _Year_ | IsSubsidiary   |
|-------------|--------|----------------|
| int         | int    | bool           |

### Suject table

| _Name_       |
|--------------|
| varchar(255) |

### Substitution table

| _SubstitutionID_   | IsPostponedEntryOrEarlyExit | TeacherID   |
|--------------------|-----------------------------|-------------|
| int                | bool                        | int         |

### Teaching table

| _TeachingID_  | Day | Hour | TeacherID | SubjectName  | CourseSection | CourseYear |
|---------------|-----|------|-----------|--------------|---------------|------------|
| int           | int | int  | int       | varchar(255) | varchar(255)  | int        |
# azure-node-test
