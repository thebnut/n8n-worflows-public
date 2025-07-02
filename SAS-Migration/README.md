# SAS-to-SQL Migration: Agentic Workflow Architecture

## Overview

This repository implements an autonomous, multi-agent system for translating SAS code to SQL using n8n workflows. The system employs a sophisticated orchestration pattern with specialised agents that collaborate to ensure accurate, reviewed, and well-documented SQL translations.

## Core Principles

### 1. **Agent Specialisation**
Each agent has a single, well-defined responsibility:
- **Coder Agent**: Expert translation from SAS to SQL
- **Reviewer Agent**: Critical evaluation and quality assurance
- **Documentation Agent**: Clear, comprehensive documentation generation
- **Orchestration Agent**: Workflow coordination and iteration management

### 2. **Iterative Refinement**
The system implements a review loop (maximum 3 iterations) that ensures:
- Initial translations are reviewed for accuracy
- Feedback is incorporated systematically
- Quality improves through multiple passes
- Graceful handling when consensus isn't reached

### 3. **Autonomous Operation**
Once initiated, the workflow runs without human intervention:
- Automatic file retrieval from GitHub
- Sequential agent execution
- Result validation and error handling
- Final output storage back to repository

## Architecture

```
┌─────────────────┐
│ GitHub Trigger  │
│  (SAS File)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Orchestration  │◄─────┐
│     Agent       │      │
└────────┬────────┘      │
         │               │
         ▼               │
┌─────────────────┐      │
│  Coder Agent    │      │
│ (SAS → SQL)     │      │
└────────┬────────┘      │
         │               │
         ▼               │
┌─────────────────┐      │
│ Reviewer Agent  │──────┘
│ (Validate SQL)  │ (Feedback Loop)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│Documentation    │
│    Agent        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ GitHub Storage  │
│ (SQL + Docs)    │
└─────────────────┘
```

## Agent Specifications

### Coder Agent
**Purpose**: Translate SAS code into functionally equivalent SQL

**Key Responsibilities**:
- Map SAS DATA steps to SQL operations
- Convert BY-group logic to window functions
- Translate SAS functions to SQL equivalents:
  - `PUT(x,zN.)` → `LPAD(CAST(x AS VARCHAR), N, '0')`
  - `COMPRESS/STRIP` → `TRIM/REGEXP_REPLACE`
  - `PRXCHANGE` → `REGEXP_REPLACE`
- Handle PROC SORT NODUP with window-function deduplication
- Maintain date/time format conversions

**Output**: Pure SQL code without explanations or markup

### Reviewer Agent
**Purpose**: Ensure SQL translation accuracy and best practices

**Review Criteria**:
1. **Functional Equivalence**: Every SAS operation properly translated
2. **Syntax Validity**: Valid ANSI-SQL (unless dialect-specific)
3. **Performance**: Efficient use of window functions and joins
4. **Maintainability**: Clear aliases, proper indentation, inline comments

**Output**: Either "Approved" or specific feedback for improvements

### Documentation Agent
**Purpose**: Generate comprehensive markdown documentation

**Documentation Includes**:
- High-level SQL purpose and objectives
- Key transformation steps explained
- Business logic and data quality filters
- Custom calculations and derivations
- Assumptions and dialect-specific notes

**Output**: Structured markdown document for stakeholders

### Orchestration Agent
**Purpose**: Coordinate the entire migration pipeline

**Workflow Management**:
1. Initial translation request to Coder Agent
2. Review loop management (max 3 iterations):
   - Send SQL to Reviewer Agent
   - If feedback received, send to Coder Agent for revision
   - Track iteration count
3. Final documentation generation
4. Output validation before storage

**Rules**:
- Sequential execution: Coder → Reviewer → Documentation
- Mandatory review (at least once)
- 3-iteration limit (accepts last attempt if not approved)
- Complete output validation (no partial results)

## Implementation Details

### Input Processing
1. SAS file retrieved from GitHub repository
2. File content extracted as plain text
3. Passed to Orchestration Agent with full context

### Memory Management
Each agent maintains session memory using:
- Custom session keys based on input content
- Buffer window memory for context retention
- Conversation history for iterative improvements

### Error Handling
- Validation at each stage
- Stop-and-error nodes for critical failures
- JSON parsing with fallback mechanisms
- Comprehensive error messages

### Output Storage
Final outputs stored in GitHub:
- SQL files: `/SQL/{filename}_{random_id}.sql`
- Documentation: `/Doc/{filename}_{random_id}.sql.doc`
- Automatic commit messages

## Best Practices

### 1. **SAS Code Preparation**
- Ensure SAS code is complete and syntactically valid
- Include all necessary DATA steps and PROCs
- Document any custom macros or functions

### 2. **Translation Guidelines**
- Preserve all business logic exactly
- Maintain data ordering where significant
- Handle null values consistently
- Keep computed columns explicit

### 3. **Review Process**
- Focus on functional correctness first
- Then optimise for performance
- Finally ensure maintainability

### 4. **Documentation Standards**
- Write for multiple audiences (developers, analysts, auditors)
- Explain the "why" not just the "what"
- Include examples for complex transformations
- Note any limitations or assumptions

## Configuration

### Model Selection
Default: GPT-3.5-turbo for orchestration, GPT-4.1 for specialist agents
- Adjust based on complexity requirements
- Consider cost vs accuracy trade-offs

### Iteration Limits
Default: 3 review iterations
- Prevents infinite loops
- Balances quality with efficiency
- Configurable in orchestration prompt

### File Naming
Pattern: `{original_name}_{4-digit-random}.{extension}`
- Prevents overwrites
- Maintains traceability
- Supports parallel executions

## Usage

1. **Setup**: Configure GitHub credentials and OpenAI API keys
2. **Execution**: Trigger workflow with target SAS filename
3. **Monitoring**: Track progress through n8n execution logs
4. **Results**: Check GitHub repository for SQL and documentation

## Benefits

- **Consistency**: Standardised translation approach
- **Quality**: Multi-stage review ensures accuracy
- **Documentation**: Automatic generation reduces manual effort
- **Scalability**: Handles multiple files autonomously
- **Traceability**: Full audit trail of translations
- **Efficiency**: Reduces migration time significantly

## Limitations

- Maximum 3 review iterations
- ANSI-SQL default (dialect-specific features need configuration)
- Requires well-structured SAS code
- Complex macros may need manual intervention

## Future Enhancements

- Support for additional SQL dialects
- Batch processing capabilities
- Performance benchmarking
- Automated testing framework
- Integration with data validation tools
