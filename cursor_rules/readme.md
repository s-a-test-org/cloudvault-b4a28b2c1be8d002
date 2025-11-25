# 🎯 #{product_name} - Cursor AI Development Guide

          Welcome to your project! This repository has been pre-configured with comprehensive AI documentation and your complete Product Requirements Document (PRD).

          ---

          ## 🚀 Quick Start - How to Use This Setup

          ### 1. **Pin This File in Cursor**
          - Open this repo in Cursor IDE (`File → Open Folder`)
          - **Pin this file** (`cursor_instruction.md`) as a **system prompt** in Cursor
          - Cursor will now understand your project context and follow these guidelines

          ### 2. **Your Project Resources**
          This repository includes:
          - 📁 **`cursor_rules/.ai/` folder** - Comprehensive development documentation
          - 📄 **`docs/prd.json`** - Your complete Product Requirements Document
          - 🏗️ **Rhino Framework** - Pre-configured backend with your data model

          ---

          ## 📚 Understanding Your cursor_rules Documentation

          ### **Main Entry Point: `cursor_rules/.ai/main.md`**
          - **Start here first** - Overview of your Rhino project architecture
          - Navigation guide to find specific documentation
          - Quick reference for common development tasks

          ### **Backend Documentation: `cursor_rules/.ai/server/`**
          Your backend is built with the **Rhino Framework**. Key files:
          - **`models.md`** - Your data models, relationships, properties, validations
          - **`policies.md`** - Authentication, authorization, role-based access
          - **`controllers.md`** - API endpoints, routing, request handling
          - **`notifications.md`** - Email/notification templates and triggers

          ### **Frontend Documentation: `cursor_rules/.ai/web/`**
          Frontend integration and customization:
          - **`component-override.md`** - How to customize Rhino UI components
          - **`rhino-api.md`** - Frontend API hooks and data fetching patterns

          ---

          ## 🎯 Your Project Context

          **Your PRD is available at**: `docs/prd.json`
          - Contains your complete data model#{' '}
          - User personas and user stories
          - Selected Rhino modules
          - Planning information from Codalio

          ---

          ## 💡 How to Generate Code with Cursor

          ### **For Backend Development:**
          1. **Read `cursor_rules/.ai/main.md` first** to understand the architecture
          2. **Check `docs/prd.json`** for your specific data requirements
          3. **Reference `cursor_rules/.ai/server/models.md`** for Rhino model conventions
          4. **Ask Cursor**:
             - "Create a new [ModelName] based on my PRD"
             - "Add API endpoints for [feature] following Rhino patterns"
             - "Set up authentication policies for [user role]"

          ### **For Frontend Development:**
          1. **Reference `cursor_rules/.ai/web/component-override.md`** for UI customization
          2. **Check `cursor_rules/.ai/web/rhino-api.md`** for data fetching patterns
          3. **Ask Cursor**:
             - "Create a custom component for [feature] using Rhino UI"
             - "Override the [ModelName] list view with custom fields"
             - "Add a dashboard showing [specific data] from my models"

          ### **For Full-Stack Features:**
          1. **Start with your PRD**: "Implement [user story] from docs/prd.json"
          2. **Follow Rhino patterns**: Cursor knows to create models → policies → controllers → UI
          3. **Reference documentation**: Cursor will check cursor_rules/.ai files for best practices

          ---

          ## 🔧 Development Workflow with Cursor

          ### **Step 1: Understand Before Building**
          ```
          "Based on my cursor_rules/.ai documentation and PRD, explain how [feature] should be implemented"
          ```

          ### **Step 2: Generate with Context**
          ```
          "Create [feature] following the patterns in cursor_rules/.ai/server/models.md and using my data model from docs/prd.json"
          ```

          ### **Step 3: Customize and Integrate**
          ```
          "Customize the UI for [feature] using the override patterns in cursor_rules/.ai/web/component-override.md"
          ```

          ---

          ## ✅ What's Pre-Configured for You

          ### **Backend (Rhino Framework)**
          - ✅ **Data models** based on your PRD planning
          - ✅ **Authentication & authorization** with policy-based access
          - ✅ **API endpoints** with automatic CRUD operations#{'  '}
          - ✅ **Database migrations** and model relationships
          - ✅ **Validation rules** and business logic patterns

          ### **Frontend (Rhino UI)**#{'  '}
          - ✅ **Pre-built components** for your models (lists, forms, detail views)
          - ✅ **Responsive design** with consistent styling
          - ✅ **Data fetching hooks** integrated with your backend
          - ✅ **Role-based UI** that respects your authentication policies

          ### **Development Tools**
          - ✅ **Comprehensive documentation** in `cursor_rules/.ai/` folder
          - ✅ **Your complete PRD** in structured JSON format
          - ✅ **Best practices** and coding patterns specific to Rhino
          - ✅ **Integration examples** and common use cases

          ---

          ## 🎪 Example Cursor Conversations

          ### **"Create a new feature"**
          ```
          User: "I want to add a commenting system to my [ModelName] based on my PRD"

          Cursor will:
          1. Check your PRD for requirements
          2. Reference cursor_rules/.ai/server/models.md for Rhino patterns
          3. Create Comment model with proper relationships
          4. Set up policies based on cursor_rules/.ai/server/policies.md
          5. Generate UI components using cursor_rules/.ai/web/ patterns
          ```

          ### **"Customize existing functionality"**
          ```
          User: "Customize the [ModelName] list view to show additional fields"

          Cursor will:
          1. Check cursor_rules/.ai/web/component-override.md for override patterns
          2. Reference your model structure from docs/prd.json
          3. Create custom component following Rhino conventions
          4. Maintain responsive design and accessibility
          ```

          ---

          ## 🚨 Important Notes

          - **Always reference your `cursor_rules/.ai/` documentation** - It contains project-specific patterns
          - **Check `docs/prd.json` first** - Your requirements are the source of truth
          - **Follow Rhino conventions** - The framework has specific patterns for maintainability
          - **Ask Cursor to explain** - If unsure, ask Cursor to walk through the cursor_rules/.ai docs first

          ---

          ## 🎯 Ready to Build?

          **Start your first conversation with Cursor:**
          ```
          "I want to understand my project structure. Please read cursor_rules/main.md and docs/prd.json, then explain what's already built and what I can customize."
          ```
         
          ## AI Development Guide

          A **Rhino Framework** project generated by [Codalio](https://codalio.com) with full AI development support.

          ## 🤖 AI-First Development

          This repository is **optimized for AI development** with Cursor IDE:

          ### **📋 Start Here: `cursor_instruction.md`**
          - **Pin this file in Cursor** as your system prompt
          - Contains complete setup and usage instructions
          - Explains how to use the `.ai/` documentation effectively

          ### **📁 AI Documentation: `cursor_rules/.ai/` folder**
          - **`.ai/main.md`** - Project overview and navigation
          - **`.ai/server/`** - Backend development patterns (Rhino Framework)
          - **`.ai/web/`** - Frontend customization and UI components

          ### **📄 Your Requirements: `docs/prd.json`**
          - Complete Product Requirements Document (in root docs folder)
          - Data models, user stories, personas
          - Pre-configured Rhino modules

          ## 🚀 Quick Start with Cursor

          1. **Open in Cursor IDE**: `File → Open Folder`
          2. **Pin system prompt**: Open `cursor_rules/cursor_instruction.md` and pin it
          3. **Start building**: Ask Cursor to explain your project structure

          ```bash
          # Example first conversation:
          "Please read cursor_rules/.ai/main.md and docs/prd.json, then explain my project structure and what I can build."
          ```

          ## 🏗️ What's Pre-Built

          This repository includes:
          - ✅ **Rhino Framework** backend with your data models
          - ✅ **Authentication & authorization** system
          - ✅ **API endpoints** for your entities
          - ✅ **Frontend components** with Rhino UI
          - ✅ **Database migrations** and relationships
          - ✅ **Comprehensive AI documentation**

          ## 📚 Learn More

          - [Rhino Framework Docs](https://www.rhino-project.org/)
          - [Cursor IDE](https://cursor.sh/)
          - [Codalio Platform](https://codalio.com)

          ---

          **Generated by Codalio** - AI-powered product development platform

          **Happy coding! 🚀**