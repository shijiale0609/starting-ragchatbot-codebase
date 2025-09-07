# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Environment Setup
```bash
# Install dependencies (requires uv package manager)
uv sync

# Create environment file from template
cp .env.example .env
# Then edit .env with your ANTHROPIC_API_KEY
```

### Running the Application
```bash
# Quick start (recommended)
chmod +x run.sh
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Development URLs
- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs

## Architecture Overview

This is a RAG (Retrieval-Augmented Generation) system that answers questions about course materials using semantic search and Claude AI.

### Core Data Flow
1. **Document Processing**: Course documents (`docs/*.txt`) are parsed by `DocumentProcessor` into structured chunks
2. **Vector Storage**: Text chunks are embedded using sentence-transformers and stored in ChromaDB via `VectorStore`
3. **Query Processing**: User queries trigger semantic search through `CourseSearchTool`, with results fed to Claude for response generation
4. **Tool-Based Architecture**: Claude uses the search tool intelligently - only searching when course-specific information is needed

### Key Components

**RAGSystem** (`rag_system.py`) - Central orchestrator that coordinates all components and manages the query processing pipeline.

**AI Generator** (`ai_generator.py`) - Manages Claude API interactions with tool-based search capabilities. Claude decides whether to search course materials or answer from general knowledge.

**Vector Store** (`vector_store.py`) - Handles ChromaDB operations including document embeddings, semantic search, and metadata filtering by course/lesson.

**Document Processor** (`document_processor.py`) - Parses structured course documents with format:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 0: Introduction
Lesson Link: [lesson_url]
[lesson content...]
```
Creates sentence-based chunks with configurable size/overlap.

**Search Tools** (`search_tools.py`) - Implements Claude's search tool interface. The `CourseSearchTool` allows filtering by course name and lesson number.

**Session Manager** (`session_manager.py`) - Maintains conversation context across queries for coherent multi-turn dialogues.

### Configuration
All settings are centralized in `config.py` including:
- Chunk size (800) and overlap (100) for document processing
- Embedding model (all-MiniLM-L6-v2) and Claude model (claude-sonnet-4-20250514)
- ChromaDB storage path and search result limits

### Document Format
Course documents must follow the structured format with metadata headers followed by lesson sections. The system extracts course metadata, parses lesson boundaries, and creates semantically meaningful chunks with contextual prefixes.

### Tool-Based Search Strategy
Claude has access to a search tool but uses it selectively:
- General knowledge questions: Answered directly without searching
- Course-specific questions: Triggers semantic search in ChromaDB
- Search results are formatted and provided as context for Claude's response
- Sources are tracked and returned to the frontend for transparency

### Frontend Integration
Simple HTML/CSS/JS interface (`frontend/`) communicates with FastAPI backend via REST endpoints (`/api/query`, `/api/courses`). Supports real-time chat with loading states and collapsible source attribution.