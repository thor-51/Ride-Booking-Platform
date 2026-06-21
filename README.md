# 🚖 Ride Booking Platform

A full-stack ride-booking application inspired by modern ride-hailing platforms like Uber and Ola. The platform enables riders to book rides, drivers to accept requests, and both parties to interact through a seamless and intuitive workflow.

---

## ✨ Features

### 👤 User Features

* Secure authentication and authorization
* Ride booking and cancellation
* Real-time ride status updates
* Fare estimation
* Ride history management

### 🚗 Captain / Driver Features

* Driver registration and login
* Accept or reject ride requests
* Ride lifecycle management
* Driver dashboard

### 🌍 Location Services

* Pickup and destination selection
* Distance-based fare calculation
* Route and location integration

---

## 🛠️ Tech Stack

### Frontend

* React
* Tailwind CSS

### Backend

* Node.js
* Express.js

### Database

* MongoDB
* Mongoose

### Authentication

* JWT Authentication
* Role-Based Access Control

### Real-Time Communication

* Socket.IO

---

## 🏗️ System Design Overview

```text
Rider App
    │
    ▼
Frontend (React)
    │
    ▼
Express API Server
    │
 ┌──┴─────────┐
 ▼            ▼
MongoDB    Socket.IO
                │
                ▼
         Driver Updates
```

---

## 🚀 Getting Started

### Clone the Repository

```bash
git clone https://github.com/thor-51/Ride-Booking.git
cd Ride-Booking
```

### Install Dependencies

Backend:

```bash
cd backend
npm install
```

Frontend:

```bash
cd frontend
npm install
```

### Configure Environment Variables

Create a `.env` file and configure:

```env
PORT=
MONGODB_URI=
JWT_SECRET=
GOOGLE_MAPS_API_KEY=
```

### Start the Application

Backend:

```bash
npm run dev
```

Frontend:

```bash
npm run dev
```

---

## 📸 Screenshots

### Landing Page

![Landing Page](assets/landing-page.png)

### User Dashboard

![Dashboard](assets/dashboard.png)

### Ride Booking Flow

![Ride Flow](assets/ride-booking-flow.png)

> Replace the screenshots above with actual project images.

---

## 🎯 Learning Outcomes

This project helped me gain hands-on experience with:

* Authentication and Authorization
* REST API Design
* Real-Time Communication
* State Management
* Database Modeling
* Full-Stack Application Development
* Scalable Backend Architecture

---

## 📬 Connect

If you have suggestions, feedback, or would like to collaborate, feel free to reach out.

⭐ If you found this project useful, consider giving it a star.
