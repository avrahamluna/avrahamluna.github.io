---
title: "Securing a Fabric Data Warehouse with RLS and CLS"
date: 2025-11-30
categories: [portfolio, fabric]
tags: [fabric, warehouse, security, rls, cls]
status: "In progress"
# icon: "/assets/icons/fabric-security.svg"
pin: false
---

Lab-style project documenting how to secure a Microsoft Fabric Warehouse
with column-level security (CLS) and row-level security (RLS).

## What it is

A walkthrough of configuring security layers so different users see only:

- The allowed **columns** (CLS)
- The allowed **rows** (RLS filters)

The project follows the official Fabric security lab and adapts it to a
hiring-manager-friendly explanation.

## Tech

- Microsoft Fabric Warehouse
- SQL security predicates
- Row-level security (RLS)
- Column-level security (CLS)

## Highlights

- Designed separate security roles for analysts vs. executives  
- Applied CLS to hide sensitive salary and PII columns  
- Implemented RLS predicates to restrict data by region / department  
- Documented the security flow with a Mermaid diagram
