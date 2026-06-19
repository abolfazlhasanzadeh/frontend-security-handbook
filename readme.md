# frontend-security-handbook

A practical, developer-first guide to frontend web security. This repository covers common attack vectors, browser security models, authentication patterns, and mitigation strategies — all with real code examples.

---

## Table of Contents

- [Chapter 1: Core Security Fundamentals](#chapter-1-core-security-fundamentals)


---

## Chapter 1: Core Security Fundamentals

> Estimated time: ~2.5 hours

This chapter covers the essential concepts every frontend developer needs to understand — origin policy, browser storage, hashing, JWT, cookies, and stateless vs. stateful authentication.

### Sections

| Section | Topic | Time |
|---------|-------|------|
| 1-1 | Understanding the Origin Concept | 2 min |
| 1-2 | Deep Dive into Origin | 8 min |
| 1-3 | Storage Mechanisms in Origin | 12 min |
| 1-4 | What is Broadcast Channel? | 6 min |
| 1-5 | Cookie Management in Origin | 10 min |
| 1-6 | What is Hash? | 4 min |
| 1-7 | Hash Lookup Concept | 9 min |
| 1-8 | Hash vs. Encoding | 7 min |
| 1-9 | Secure Password Storage with Hashing | 12 min |
| 1-10 | Rainbow Table Attack | 8 min |
| 1-11 | What is Ciphertext? | 10 min |
| 1-12 | Interpreted vs. Compiled | 8 min |
| 1-13 | Stateless vs. Stateful | 6 min |
| 1-14 | Login System with JWT | 12 min |
| 1-15 | Deep Dive into Stateless Server | 17 min |
| 1-16 | JWT vs. JWE | 8 min |
| 1-17 | Deep Dive into Cookie | 14 min |
| 1-18 | SameSite Cookie Attribute | 7 min |

---

### What You'll Learn

**Origin & Same-Origin Policy**
How browsers enforce origin-based restrictions, when CORS is needed, and how to handle cross-origin requests properly.

**Storage Security**
The security implications of localStorage, sessionStorage, cookies, and IndexedDB. Which storage types are safe for sensitive data and why.

**Hashing & Encoding**
The difference between hashing and encoding, when to use each, and why password hashing belongs on the server.

**Authentication Patterns**
Stateless vs. stateful authentication, JWT structure and secure storage, cookie attributes (HttpOnly, Secure, SameSite), and refresh token flows.

---

[Read Chapter 1 →](./core-security-fundamentals.md)

---

## Who This Is For

Frontend developers who want to understand security from a practical perspective. Full-stack engineers looking to bridge the gap between client and server security. Tech leads implementing security standards across teams.

---

## How to Use This Guide

Start with Chapter 1 to build a solid foundation. Move through the chapters in order, or jump to specific topics based on your needs. Each chapter includes time estimates and code examples you can run locally.

---

## Prerequisites

Basic JavaScript knowledge. Familiarity with React or similar frameworks is helpful but not required.

---

## License

MIT