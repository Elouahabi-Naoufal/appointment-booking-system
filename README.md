<!-- Improved compatibility of back to top link: See: https://github.com/othneildrew/Best-README-Template/pull/73 -->
<a id="readme-top"></a>

<!-- PROJECT SHIELDS -->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![License][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

<!-- PROJECT TITLE -->
<br />
<div align="center">
  <h3 align="center">Appointment Booking System</h3>

  <p align="center">
    Spring Boot services for medical appointments and patient records, built as part of a
    distributed-systems course project.
    <br />
    <a href="https://github.com/Elouahabi-Naoufal/appointment-booking-system"><strong>Explore the repository »</strong></a>
    <br />
    <br />
    <a href="https://github.com/Elouahabi-Naoufal/appointment-booking-system/issues">Report Bug</a>
    &middot;
    <a href="https://github.com/Elouahabi-Naoufal/appointment-booking-system/issues">Request Feature</a>
  </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#services-in-this-repository">Services in this repository</a></li>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->
## About The Project

This repository contains two independently runnable Spring Boot services for a medical
appointment-booking system. Each service owns its own MariaDB database, exposes a REST API and calls
the other services over HTTP with `RestTemplate`.

### Services in this repository

| Module | Port | Database | Responsibility |
| --- | --- | --- | --- |
| `rendezvous-service` | `8083` | `rendezvous_db` | Create, list, inspect, cancel and change the status of appointments; store notes |
| `dossiers-service` | `8084` | `dossiers_db` | Medical records and consultation reports (`comptes-rendus`) |

`rendezvous-service` validates a patient, doctor and time slot before saving an appointment, then
blocks the slot and emits a notification event. Cancelling an appointment frees the slot.
`dossiers-service` validates the patient, appointment and doctor before creating a record or report,
and can finalize a report.

Important limitations of the current state:

* The course subject describes five services (patients, doctors, planning, appointments and
  notifications/history). **Only the appointment and records services are implemented here.**
* Validation calls are **fail-open**: if a downstream service is unreachable, `MicroserviceClient`
  logs a warning and treats the check as successful. The services therefore run without the missing
  modules, but cross-service validation is not enforced.
* There is no parent `pom.xml`; build and run each module separately.
* No automated tests are committed.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Built With

* [![Java][java-shield]][java-url] — Java 25
* [![Spring Boot][spring-shield]][spring-url] — Spring Boot 3.4.4
* [![Spring Data JPA][jpa-shield]][jpa-url]
* [![MariaDB][mariadb-shield]][mariadb-url]
* [![Maven][maven-shield]][maven-url]
* [![OpenAPI][openapi-shield]][openapi-url] — springdoc, Swagger UI

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- GETTING STARTED -->
## Getting Started

### Prerequisites

* JDK 25
  ```sh
  java -version
  ```
* Maven 3.9+
  ```sh
  mvn -version
  ```
* A running MariaDB server. The default connection is
  `jdbc:mariadb://localhost:3306/<db>?createDatabaseIfNotExist=true` with user `root`.
* The other services are optional at runtime because validation fails open, but full validation
  needs them on ports `8081` (patients), `8082` (doctors), `8085` (notifications) and `8086`
  (planning).

### Installation

1. Clone the repository
   ```sh
   git clone https://github.com/Elouahabi-Naoufal/appointment-booking-system.git
   cd appointment-booking-system
   ```
2. Start MariaDB and make sure the configured user can create the two databases.
3. Configure the connection with environment variables instead of editing the properties files:
   ```sh
   export DB_URL='jdbc:mariadb://localhost:3306/rendezvous_db?createDatabaseIfNotExist=true'
   export DB_USERNAME='your-user'
   export DB_PASSWORD='your-password'
   ```
4. Build and run the appointment service
   ```sh
   cd rendezvous-service
   mvn spring-boot:run
   ```
5. In another terminal, run the records service
   ```sh
   cd dossiers-service
   mvn spring-boot:run
   ```

Both modules use `spring.jpa.hibernate.ddl-auto=update`, so tables are created from the JPA entities
on first start.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- USAGE -->
## Usage

Swagger UI is available at `http://localhost:8083/swagger-ui.html` and
`http://localhost:8084/swagger-ui.html`. The OpenAPI document is served at `/api-docs`.

Appointment endpoints (`rendezvous-service`):

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/api/rendezvous` | Create an appointment |
| `GET` | `/api/rendezvous` | List all appointments |
| `GET` | `/api/rendezvous/{id}` | Get one appointment |
| `GET` | `/api/rendezvous/patient/{patientId}` | Appointments for a patient |
| `GET` | `/api/rendezvous/medecin/{medecinId}` | Appointments for a doctor |
| `PUT` | `/api/rendezvous/{id}/statut` | Change status (`EN_ATTENTE`, `CONFIRME`, `ANNULE`, `TERMINE`) |
| `PUT` | `/api/rendezvous/{id}/annuler` | Cancel and free the slot |
| `PUT` | `/api/rendezvous/{id}/notes` | Update notes |

Example request:

```sh
curl -X POST http://localhost:8083/api/rendezvous \
  -H 'Content-Type: application/json' \
  -d '{
    "patientId": 1,
    "medecinId": 2,
    "creneauId": 10,
    "dateHeureRendezvous": "2026-04-20T10:30:00",
    "motif": "Consultation"
  }'
```

Records endpoints (`dossiers-service`) live under `/api/dossiers` and `/api/comptes-rendus`; both
support create, list, read, update and delete, and reports expose `/finaliser`.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- ROADMAP -->
## Roadmap

- [x] Appointment service with slot blocking, cancellation and status changes
- [x] Medical-record and consultation-report service
- [x] Swagger UI for both services
- [ ] Implement the missing patients, doctors, planning and notifications/history services
- [ ] Replace fail-open validation with explicit availability checks and circuit breaking
- [ ] Add a parent build and a Docker Compose stack for MariaDB plus all services
- [ ] Add integration tests for the cross-service flows
- [ ] Remove committed `target/` build output and move secrets out of `.env` files

See the [open issues](https://github.com/Elouahabi-Naoufal/appointment-booking-system/issues) for a full list of proposed features and known issues.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTRIBUTING -->
## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please fork the repo and create a pull request. You can also simply open an issue with the tag "enhancement".
Don't forget to give the project a star! Thanks again!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Top contributors:

<a href="https://github.com/Elouahabi-Naoufal/appointment-booking-system/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=Elouahabi-Naoufal/appointment-booking-system" alt="contrib.rocks image" />
</a>

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- LICENSE -->
## License

No `LICENSE` file is bundled with this repository. This project is not licensed — all rights reserved.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTACT -->
## Contact

Naoufal Elouahabi
* LinkedIn: [https://www.linkedin.com/in/elouahabi-naoufal/](https://www.linkedin.com/in/elouahabi-naoufal/)
* Email: [elouahabi-naoufal1@gmail.com](mailto:elouahabi-naoufal1@gmail.com)

Project Link: [https://github.com/Elouahabi-Naoufal/appointment-booking-system](https://github.com/Elouahabi-Naoufal/appointment-booking-system)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- ACKNOWLEDGMENTS -->
## Acknowledgments

* [Spring Boot](https://spring.io/projects/spring-boot)
* [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
* [MariaDB](https://mariadb.org/)
* [springdoc-openapi](https://springdoc.org/)
* [Best-README-Template](https://github.com/othneildrew/Best-README-Template)
* [Img Shields](https://shields.io)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/Elouahabi-Naoufal/appointment-booking-system.svg?style=for-the-badge
[contributors-url]: https://github.com/Elouahabi-Naoufal/appointment-booking-system/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/Elouahabi-Naoufal/appointment-booking-system.svg?style=for-the-badge
[forks-url]: https://github.com/Elouahabi-Naoufal/appointment-booking-system/network/members
[stars-shield]: https://img.shields.io/github/stars/Elouahabi-Naoufal/appointment-booking-system.svg?style=for-the-badge
[stars-url]: https://github.com/Elouahabi-Naoufal/appointment-booking-system/stargazers
[issues-shield]: https://img.shields.io/github/issues/Elouahabi-Naoufal/appointment-booking-system.svg?style=for-the-badge
[issues-url]: https://github.com/Elouahabi-Naoufal/appointment-booking-system/issues
[license-shield]: https://img.shields.io/badge/License-All%20rights%20reserved-red.svg?style=for-the-badge
[license-url]: https://github.com/Elouahabi-Naoufal/appointment-booking-system
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/elouahabi-naoufal/
[java-shield]: https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white
[java-url]: https://openjdk.org/
[spring-shield]: https://img.shields.io/badge/Spring%20Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white
[spring-url]: https://spring.io/projects/spring-boot
[jpa-shield]: https://img.shields.io/badge/Spring%20Data%20JPA-6DB33F?style=for-the-badge&logo=spring&logoColor=white
[jpa-url]: https://spring.io/projects/spring-data-jpa
[mariadb-shield]: https://img.shields.io/badge/MariaDB-003545?style=for-the-badge&logo=mariadb&logoColor=white
[mariadb-url]: https://mariadb.org/
[maven-shield]: https://img.shields.io/badge/Apache%20Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white
[maven-url]: https://maven.apache.org/
[openapi-shield]: https://img.shields.io/badge/OpenAPI-6BA539?style=for-the-badge&logo=openapiinitiative&logoColor=white
[openapi-url]: https://springdoc.org/
