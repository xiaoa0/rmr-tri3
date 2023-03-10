---
toc: true
comments: false
layout: post
title:  Spring Backend, Database, Notes, Admin Frontend
description: Some tips on building a Notes database using relationships.
categories: []
type: pbl
week: 18
---


## Building Database Relationships
> This blog is built off of information developed for last years class.  There are a couple of Database relationships in the project. The UIs are build using ***Thymeleaf which will `NOT` be used*** this year, unless you want to build and Admin console to your backend Spring project. Here are some key links ...
- Relations Theme Videos [We are the World](https://youtu.be/Glny4jSciVI?t=241), [We are Family](https://youtu.be/oMVe_HcyP9Y?t=29)
- Access... tedison@example.com  123toby
- ***Featured*** [Building Notes for Person](https://csa.nighthawkcodingsociety.com/database/person)
- [Building Teams from Persons](https://csa.nighthawkcodingsociety.com/database/scrum)
- GitHub code for [Database](https://github.com/nighthawkcoders/nighthawk_csa/tree/master/src/main/java/com/nighthawk/csa/mvc/database)
- GitHub code for [Thymeleaf](https://github.com/nighthawkcoders/nighthawk_csa/tree/master/src/main/resources/templates/mvc/database)


### Dependencies 
> Review [POM file](https://github.com/nighthawkcoders/nighthawk_csa/blob/master/pom.xml) from GitHub Project that use features mentioned.  The Notes in this project does include support for Markdown.
 
```text
<dependency>
    <groupId>org.commonmark</groupId>
    <artifactId>commonmark</artifactId>
    <version>0.17.2</version>
</dependency>
```

### POJO - Define Note table and Many-to-One relationship
POJO is the foundation for table definition.  Key `NEW` parts are described...
* The @ManyToOne annotation establishes relationship between this Note record and Person to which it belongs.
* The @Column(columnDefinition="TEXT") enables SQL to define this as a very large (Blob) style database type.  This is not restricted in the database like a ordinary string.
* FYI, @Data is shortcut, it is like having @Getter, @Setter, @ToString, @EqualsAndHashCode and @RequiredArgsConstructor annotations on the class.

```java
package com.nighthawk.csa.mvc.database.note;

import com.nighthawk.csa.mvc.database.person.Person;
import lombok.*;

import javax.persistence.*;
import javax.validation.constraints.NotNull;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class Note {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @ManyToOne
    @JoinColumn(name="person_id")
    private Person person;

    @NotNull
    @Column(columnDefinition="TEXT")
    private String text;
}
```


### JPA - Find all Notes with Person Object
This Class extends JPA which provides standard database queries.  There is one additional method defined using JPA interface definitions, findAllByPerson which returns a List<Note> which is all notes for a Person.

```java
package com.nighthawk.csa.mvc.database.note;

import com.nighthawk.csa.mvc.database.person.Person;
import com.nighthawk.csa.mvc.database.role.Role;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface NoteJpaRepository extends JpaRepository<Note, Long> {
    List<Note> findAllByPerson(Person p);
}
```


### Controller - View Page
This Control method focuses on loading data for page.  Key logic is shared...
* This form needs to be called with valid `id`, this is achieved by loading through table
* Person object is loaded through its `id`
* List<Note> object is loaded with ***JPA query*** which takes a `Person object` as parameter
* Note an `object is prepared` for ***Save*** operation in UI
* Very important, the convertMarkdownToHTML method is built to change the saved text/markdown/html to HTML and has method "public static String convertMarkdownToHTML(String markdown)" and multiple import org.commonmark.* dependencies.  See [NoteViewController.java](https://github.com/nighthawkcoders/nighthawk_csa/blob/master/src/main/java/com/nighthawk/csa/mvc/database/note/NoteViewController.java)

```java
    @GetMapping("/database/notes/{id}")
    public String notes(@PathVariable("id") Long id, Model model) {
        Person person = modelRepository.get(id);
        List<Note> notes = noteRepository.findAllByPerson(person);
        Note note = new Note();
        note.setPerson(person);

        for (Note n : notes)
            n.setText(convertMarkdownToHTML(n.getText()));

        model.addAttribute("person", person);
        model.addAttribute("notes", notes);
        model.addAttribute("note", note);
        return "mvc/database/notes";
    }
```


### Controller - Save Note via JPA method
This Control method focuses on ***saving the Note***.  Key logic is shared...
* After this method is done Error or Success, it needs to return to Notes page, the String redirect = "redirect:/database/notes/"+note.getPerson().getId() is to make sure it goes back to same Person
* Note is ***saved via JPA***: noteRepository.save(note);

```java
    @PostMapping("/database/notes")
    public String notesAdd(@Valid Note note, BindingResult bindingResult) {
        // back to person ID on redirect
        String redirect = "redirect:/database/notes/"+note.getPerson().getId();

        // database errors
        if (bindingResult.hasErrors()) {
            return redirect;
        }

        // note is saved and person ID is pre-set
        noteRepository.save(note);

        // Redirect to next step
        return redirect;
    }
```


### Frontend - Form and Action for New Note
The Form is used for new Post/Blog.  Here are key elements...
* The form is bound using bound to the note object (see th:object="${note}"). 
* It is critical to have th:field="*{person.id}" hidden reference.  This will link Note to Person in backend save.
* The textarea is used for input to be more Note like.

```html
   <form class="form-inline" action="#" th:action="@{/database/notes}" th:object="${note}" method="POST">
        <table class="table">
          <thead>
          <tr>
            <!-- avoid warnings on binding person.id line -->
            <label for="id" hidden>ID</label>
          </tr>
          <tr>
            <th><label for="text">Create a new note:</label></th>
          </tr>
          </thead>
          <tbody>
            <tr>
              <!-- everything fails without binding person.id -->
              <input type="number" th:field="*{person.id}" id="id" hidden class="form-control-plaintext" >
            </tr>
            <tr>
              <td><textarea rows = "5" cols = "100%" th:field="*{text}" id="text" required></textarea></td>
            <td><input type="submit" value="Add" ></td>
          </tr>
          </tbody>
        </table>
      </form>
```


### Frontend - Display Existing Notes
A table is used to display history of Post/Blog.  Here are key elements...
* The "notes" is received as a data source that is specific to the logged in User
* Uses th:each to process each element in list
* UI preference is these are in reverse order and array is used to process them backwards (choice is to handle this in frontend but it could be handled in JPA query as well.
* Very import "th:utext" is used to make sure it is rendered HTML versus text-based HTML

```html
    <div class="table-responsive">
        <table class="table">
          <tbody>
            <!--Notes output area, Thymeleaf lines iterate through notes backwards,
                note.text is html converted from markdown requires th:utext to render correctly
            -->
            <tr th:each="i : ${#numbers.sequence(notes.size() - 1, 0, -1)}"
                th:with="n=${notes[i]}">
              <td><span th:utext="${n.text}"></span></td>
            </tr>
          </tbody>
        </table>
      </div>
```


## Hacks
This blog adds to the concept of persistent data for a Person POJO.   As it is focused on building a relational table for Notes to Person.  Notes supports text/markdown/html rendering.  [Annotations and Hibernate](https://www.digitalocean.com/community/tutorials/jpa-hibernate-annotations) are used.  Explore and build other options for your tables and relationships.  FYI, there is even inheritance in POJOs.  You will learn a lot about Java if you explore with true PBL mindset.

* Requirement for Project
    * Make sure that Database is constructed
    * Have many tables in the Database
    * Build relationships in tables, ie Person has many Notes
    * Establish Roles in accessing data (ie Delete is only done by ADMIN)

* Expectations of learnings form this Tech Talk
    * Create a POJO with Many-to-One relationship 
    * Build JPA to access all elements from and ID
    * Establish Controller methods and APIs, Build Frontend with JavaScript
    * Extra Credit, Build Admin Frontend with Thymeleaf
