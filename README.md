# ACE Developer Mode

## Overview

The **ACE Developer** mode is a custom mode for IBM Bob that enables enterprise-grade development of IBM App Connect Enterprise (ACE) v13 integration solutions. This mode combines IBM's ace-bob skill with enterprise-specific best practices, patterns, and guidelines to ensure all generated ACE artifacts align with organizational standards.

## Purpose

This custom mode demonstrates how enterprises can:

1. **Leverage IBM's ace-bob Skill** - Use the authoritative ACE artifact generation capabilities provided by IBM
2. **Integrate Enterprise Best Practices** - Layer organizational standards, patterns, and guidelines on top of IBM's foundation
3. **Ensure Consistency** - Generate ACE integrations that consistently follow enterprise architecture and coding standards
4. **Accelerate Development** - Provide developers with proven patterns for common integration scenarios
5. **Maintain Quality** - Enforce error handling, audit logging, and operational excellence patterns

## ace-bob Skill

This mode uses the **ace-bob** skill published at:
**https://github.com/ot4i/ace-bob**

The ace-bob skill provides:
- Authoritative definitions for ACE Toolkit project structures
- Correct xmi:type namespace prefixes for all ACE node types
- Eclipse project requirements and configurations
- Best practices for ESQL, Java, and message flow development
- Guidance for specific connectors (Salesforce, ServiceNow, AWS, Azure, etc.)

### Why ace-bob is Critical

The ace-bob skill ensures that all generated ACE artifacts:
- Are compatible with ACE Toolkit v13
- Use correct XML namespaces and node types
- Follow IBM's recommended project structures
- Can be imported and edited in ACE Toolkit without errors

**Important:** This mode ALWAYS uses the ace-bob skill for creating .msgflow, .esql, .java, .map, and .subflow files to ensure technical correctness.

## Enterprise Customization

While ace-bob provides the technical foundation, this custom mode adds enterprise-specific layers:

### 1. Enterprise Patterns (6_enterprise_patterns.xml)
Derived from real-world production implementations, including:
- **Audit Logging Pattern** - Standardized message tracking across all flows
- **Error Handling Pattern** - Centralized error capture with email notifications
- **File Processing Pattern** - Dynamic naming, archival, and error handling
- **Database Integration Pattern** - Connection pooling, transaction management
- **Deployment Pattern** - BAR file creation and environment promotion

### 2. ESQL Best Practices (2_esql_best_practices.xml)
Enterprise coding standards including:
- Performance optimization techniques (REFERENCE usage, tree copying minimization)
- Memory management guidelines
- Loop optimization patterns
- String manipulation efficiency
- Field existence checking methods
- Common utility procedures
- Documentation standards

### 3. Message Flow Patterns (4_message_flow_patterns.xml)
Proven flow designs for:
- Request-reply HTTP services
- MQ-to-MQ transformations
- File-based integration
- Database polling
- Aggregation and scatter-gather patterns
- Subflow reusability patterns

### 4. Project Structure Standards (3_project_structure_patterns.xml)
Organizational conventions for:
- Project naming and organization
- Directory structures
- Properties file management
- Dependency management
- Version control practices
- BAR file organization

### 5. Development Workflow (1_workflow.xml)
Step-by-step guidance through:
- Requirements analysis
- Project creation
- Message flow design
- ESQL development
- Audit implementation
- Testing and validation
- Deployment preparation

### 6. ace-bob Integration Guide (5_ace_bob_skill_integration.xml)
Detailed instructions on:
- When and how to use the ace-bob skill
- Node type lookup process
- Application Connector configuration
- Troubleshooting common issues
- Integration with enterprise patterns

## How It Works

When you use the ACE Developer mode:

1. **IBM Bob analyzes your request** - Understands what ACE artifacts you need
2. **Consults enterprise patterns** - Checks for similar implementations and best practices
3. **Leverages ace-bob skill** - Uses IBM's skill to generate technically correct artifacts
4. **Applies enterprise standards** - Ensures error handling, audit logging, and naming conventions
5. **Validates completeness** - Confirms all required files and configurations are in place

## Example Use Cases

### Creating a New Integration Flow
```
User: "Create an ACE flow that reads from an MQ queue, transforms the message, 
       and calls a REST API"

ACE Developer Mode:
1. Creates ACE Application project with correct structure
2. Generates message flow with MQ Input, Compute, HTTP Request nodes
3. Creates ESQL with audit initialization and error handling
4. Adds error handling subflow with email notification
5. Creates properties files for all environments
6. Includes comprehensive documentation
```

### Implementing Error Handling
```
User: "Add enterprise-standard error handling to my flow"

ACE Developer Mode:
1. References enterprise error handling pattern
2. Adds Catch terminal connection
3. Invokes error handling subflow
4. Configures email notification
5. Updates audit parameters
6. Follows established patterns from production flows
```

### Creating Reusable Components
```
User: "Create a subflow for database audit logging"

ACE Developer Mode:
1. Creates Library project if needed
2. Generates subflow with standard audit fields
3. Implements database insert with error handling
4. Adds to shared library for reuse
5. Documents usage and dependencies
```

## Benefits

### For Developers
- **Faster Development** - Proven patterns and templates accelerate coding
- **Consistency** - All code follows the same standards
- **Learning** - Built-in best practices teach proper ACE development
- **Quality** - Automated inclusion of error handling and audit logging

### For Architects
- **Standards Enforcement** - Ensures architectural guidelines are followed
- **Pattern Reuse** - Promotes consistent solutions across teams
- **Maintainability** - Standardized code is easier to maintain
- **Knowledge Capture** - Preserves organizational expertise

### For Operations
- **Reliability** - Comprehensive error handling and monitoring
- **Troubleshooting** - Standardized audit logging aids diagnosis
- **Deployment** - Consistent BAR file structure and properties management
- **Support** - Email notifications alert teams to issues

## File Structure

```
.bob/rules-ace-developer/
├── README.md                           # This file
├── 1_workflow.xml                      # Development workflow guidance
├── 2_esql_best_practices.xml          # ESQL coding standards
├── 3_project_structure_patterns.xml   # Project organization
├── 4_message_flow_patterns.xml        # Flow design patterns
├── 5_ace_bob_skill_integration.xml    # ace-bob usage guide
└── 6_enterprise_patterns.xml          # Real-world patterns

.bobmodes                               # Mode configuration
```

## Getting Started

### Prerequisites

1. **Download the ace-bob mode** - Clone or download this repository to your local machine
2. **Install the ace-bob skill** - The ace-bob skill must be checked out from GitHub:
   ```bash
   cd .bob/skills/
   git clone https://github.com/ot4i/ace-bob.git
   ```
   This ensures you have the latest version of IBM's ace-bob skill with all ACE artifact generation capabilities.

### Using the Mode

1. **Switch to ACE Developer mode** when working on ACE projects
2. **Describe your integration requirement** - Bob will guide you through the process
3. **Review generated artifacts** - All files follow enterprise standards
4. **Import into ACE Toolkit** - Use File > Import > Existing Projects into Workspace

## Example Commands

```
"Create a new ACE application for processing customer orders from MQ"

"Add error handling with email notification to my flow"

"Create a subflow for audit logging"

"Generate ESQL to transform XML to JSON"

"Create properties files for all environments"

"Build a file processing flow with dynamic naming"
```

## Customization

Enterprises can customize this mode by:

1. **Updating enterprise patterns** - Add your organization's specific patterns
2. **Modifying naming conventions** - Adjust to match your standards
3. **Adding custom utilities** - Include organization-specific procedures
4. **Configuring properties** - Set default values for your environments
5. **Extending workflows** - Add organization-specific steps

## Technical Requirements

- IBM App Connect Enterprise v13 (ACE Toolkit)
- IBM Bob with ace-bob skill installed
- Access to example ACE projects (for pattern learning)
- Understanding of enterprise integration patterns

## Support and Maintenance

This mode is designed to evolve with your organization:

- **Add new patterns** as they emerge from production implementations
- **Update best practices** based on lessons learned
- **Refine workflows** to match changing processes
- **Extend coverage** to new integration scenarios

## Acknowledgments

- **IBM ace-bob skill** - Foundation for ACE artifact generation (https://github.com/ot4i/ace-bob)
- **Enterprise ACE teams** - Real-world patterns and best practices
- **IBM App Connect Enterprise** - Integration platform

## License

This custom mode builds upon IBM's ace-bob skill and is intended for enterprise use in conjunction with IBM App Connect Enterprise.

---

**Note:** This mode demonstrates the power of combining IBM's official skills with enterprise-specific knowledge to create a tailored development experience that ensures quality, consistency, and alignment with organizational standards.
