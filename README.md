# Exp-04-Spring-Boot-with-REST-API-and-Hibernate-Integration

## AIM:

To develop a Spring Boot application to store and retrieve data from a Movies database using Object Relational Mapping (ORM) with Hibernate and expose it via REST APIs.

## ALGORITHM:

Create Spring Boot project with dependencies:

- Spring Web
- Spring Data JPA
- H2 or MySQL Database

Configure application.properties with DB connection and JPA settings.

Create Movie entity with fields like id, title, genre, rating, and year.

Create MovieRepository interface extending JpaRepository.

Create MovieController to define REST endpoints for CRUD operations:

- GET /movies
- GET /movies/{id}
- POST /movies
- PUT /movies/{id}
- DELETE /movies/{id}

## Program:

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>4.1.1</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>movies</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name/>
	<description/>
	<url/>
	<licenses>
		<license/>
	</licenses>
	<developers>
		<developer/>
	</developers>
	<scm>
		<connection/>
		<developerConnection/>
		<tag/>
		<url/>
	</scm>
	<properties>
		<java.version>26</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-h2console</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webmvc</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webmvc-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

### application.properties

```properties
spring.application.name=movies

# H2 Database
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# Hibernate / JPA
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# H2 Console
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

```

### Movie.java

```java
package com.example.movies.model;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Movie {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    private String genre;

    @Column(name = "release_year")
    private int releaseYear;

    private double rating;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getGenre() {
        return genre;
    }

    public void setGenre(String genre) {
        this.genre = genre;
    }

    public int getReleaseYear() {
        return releaseYear;
    }

    public void setReleaseYear(int releaseYear) {
        this.releaseYear = releaseYear;
    }

    public double getRating() {
        return rating;
    }

    public void setRating(double rating) {
        this.rating = rating;
    }
}

```

### MovieRepository.java

```java
package com.example.movies.repository;
import com.example.movies.model.Movie;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
@Repository
public interface MovieRepository extends JpaRepository<Movie, Long> {

}

```

### MovieController.java

```java
package com.example.movies.controller;

import com.example.movies.model.Movie;
import com.example.movies.repository.MovieRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/movies")
public class MovieController {

    @Autowired
    private MovieRepository repo;

    // GET all movies
    @GetMapping
    public List<Movie> getAllMovies() {
        return repo.findAll();
    }

    // GET movie by ID
    @GetMapping("/{id}")
    public ResponseEntity<Movie> getMovieById(@PathVariable Long id) {

        return repo.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // POST movie
    @PostMapping
    public Movie addMovie(@RequestBody Movie movie) {
        return repo.save(movie);
    }

    // PUT movie
    @PutMapping("/{id}")
    public ResponseEntity<Movie> updateMovie(
            @PathVariable Long id,
            @RequestBody Movie movieDetails) {

        return repo.findById(id).map(movie -> {

            movie.setTitle(movieDetails.getTitle());
            movie.setGenre(movieDetails.getGenre());
            movie.setReleaseYear(movieDetails.getReleaseYear());
            movie.setRating(movieDetails.getRating());

            return ResponseEntity.ok(repo.save(movie));

        }).orElse(ResponseEntity.notFound().build());
    }

    // DELETE movie
    @DeleteMapping("/{id}")
    public ResponseEntity<Object> deleteMovie(@PathVariable Long id) {

        return repo.findById(id).map(movie -> {

            repo.delete(movie);
            return ResponseEntity.ok().build();

        }).orElse(ResponseEntity.notFound().build());
    }
}

```

## Output

### POST /movies

<img width="1269" height="791" alt="Screenshot 2026-09-07 103842" src="https://github.com/user-attachments/assets/e1922b4c-4372-40f7-8f5e-2b722b35aca0" />


### GET /movies

<img width="1278" height="796" alt="Screenshot 2026-09-07 103914" src="https://github.com/user-attachments/assets/2ae73b80-2336-4eba-8b42-3a8c78df347f" />


### PUT /movies/{id}

<img width="1276" height="800" alt="Screenshot 2026-09-07 103945" src="https://github.com/user-attachments/assets/7169ca8b-2002-47f9-8e26-2f575e7aff34" />


### DELETE /movies/{id}

<img width="1277" height="798" alt="Screenshot 2026-09-07 104009" src="https://github.com/user-attachments/assets/9e12a948-dbc3-4f06-b2ff-17b6daf0a0f3" />


## Result

Thus the development of a Spring Boot application to store and retrieve data from a Movies database is completed successfully
