# 🍽️ FoodFrenzy

FoodFrenzy is a comprehensive **restaurant/order management system** designed for managing **customers, inventory, and orders**.  
It offers **secure authentication**, **role-based access control**, and **MySQL integration**.  
Built with **Spring Boot and Thymeleaf**, the application provides a seamless experience for **admins and staff**.  

📌 *Note*: This project is a **forked repository**, which I enhanced by adding a **complete CI/CD + DevSecOps pipeline** and **cloud-native monitoring**.  

![Screenshot](https://github.com/user-attachments/assets/1382d32f-3cbb-40c3-b6b5-9fc55cd5176f)  

---

## 🚀 Features  

- **Customer Management**: Add, update, and delete customer information.  
- **Inventory Management**: Track stock levels and pricing.  
- **Order Management**: Create, update, and manage customer orders.  
- **User Authentication**: Secure login for admins and staff.  
- **Role-Based Access Control**: Define roles & permissions.  
- **Dynamic UI**: Built with **Thymeleaf templates**.  
- **Database Integration**: Powered by **MySQL**.  

---

## 🛠️ Technology Stack  

- **Backend**: Spring Boot, Java 8, Spring MVC, Spring Data JPA (Hibernate)  
- **Frontend**: Thymeleaf, HTML, CSS, JavaScript  
- **Database**: MySQL  
- **Dev Tools**: Eclipse / Spring Tool Suite (STS), Maven  

---

## ⚡ DevOps & CI/CD Enhancements  

I extended this project with a **production-style DevOps pipeline**:  

- **Jenkins (on AWS EC2)** for automated builds.  
- **Java 17 & Maven 3.9.11** for build management.  
- **Trivy** for container vulnerability scanning.  
- **OWASP Dependency Check** for dependency scanning.  
- **SonarQube** for static code analysis & quality gates.  
- **Docker & Docker Hub** for image build and distribution.  
- **Docker Compose** for application + MySQL deployment.  
- **Prometheus + Grafana** for real-time monitoring.  

📌 These enhancements ensure **automation, security, and monitoring**, making the project closer to a **real-world DevSecOps workflow**.  

---

## 📦 Prerequisites  

- Java 8  
- MySQL  
- Maven  
- Eclipse / Spring Tool Suite (STS)  

---

## ⚙️ Setup and Installation  

1. Clone the repository:  
   ```bash
   git clone https://github.com/your-repository-url/FoodFrenzy.git
   cd FoodFrenzy
   ```

2. Configure MySQL Database in `application.properties`:  
   ```properties
   spring.datasource.url=jdbc:mysql://localhost:3306/foodfrenzy
   spring.datasource.username=root
   spring.datasource.password=root
   spring.jpa.hibernate.ddl-auto=update
   ```

3. Run the project:  
   ```bash
   mvn spring-boot:run
   ```

4. Access the application:  
   ```
   http://localhost:8080
   ```

---


