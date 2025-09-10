---
description: 'Instructions for developing Microsoft 365 Copilot declarative agents using schema v1.5 with Microsoft 365 Agents Toolkit integration'
---

# Comprehensive Development Guide for Microsoft 365 Copilot Declarative Agents

## Microsoft 365 Agents Toolkit Integration

These instructions are optimized for use with the **Microsoft 365 Agents Toolkit** VS Code extension (`teamsdevapp.ms-teams-vscode-extension`), which provides:

### Toolkit Features
- **Project Templates**: Create new declarative agent projects from built-in templates
- **TypeSpec Support**: Define agents using TypeSpec that compiles to JSON manifests
- **Local Debugging**: Test agents locally with Agents Playground
- **Lifecycle Management**: Create → Debug → Provision → Deploy workflow
- **Environment Management**: Multi-environment support (.env.dev, .env.prod)
- **Manifest Validation**: Built-in schema validation and CodeLens previews

### Installation
```vscode-extensions
teamsdevapp.ms-teams-vscode-extension
```

### Quick Start with Toolkit
1. Open VS Code with Agents Toolkit installed
2. Select **Create a New Agent/App** → **Teams App** → **Agent for Copilot**
3. Choose **Declarative Agent** template
4. Select programming language (TypeScript/JavaScript)
5. Configure project location and name

---

You are an expert in developing declarative agents for Microsoft 365 Copilot using the v1.5 schema specification. Follow these comprehensive guidelines when creating, reviewing, or modifying declarative agent manifests.

## TypeSpec Agent Definitions (Toolkit Preferred)

The Microsoft 365 Agents Toolkit uses TypeSpec for agent definition, which provides better type safety and developer experience.

### Basic TypeSpec Structure
```typescript
import "@microsoft/m365-agents-typespec";
using AgentCapabilities;

@instructions("""
  Your agent instructions here.
  Use triple quotes for multi-line instructions.
""")
@conversationStarter(#{
  title: "Starter 1",
  text: "How can I help with..."
})
@conversationStarter(#{
  title: "Starter 2", 
  text: "What would you like to know about..."
})
namespace YourAgentName {
  // Capability definitions go here
}
```

### Common TypeSpec Capability Patterns

**Web Search**
```typescript
op webSearch is AgentCapabilities.WebSearch<
  TSites = [{
    url: "https://learn.microsoft.com"
  }]
>;
```

**OneDrive and SharePoint**
```typescript
op oneDriveAndSharePoint is AgentCapabilities.OneDriveAndSharePoint<
  TItems = [{
    sharepoint_site_id: "${{SHAREPOINT_SITE_ID}}",
    sharepoint_list_id: "${{SHAREPOINT_LIST_ID}}"
  }]
>;
```

**Multiple Capabilities**
```typescript
namespace ComprehensiveAgent {
  op webSearch is AgentCapabilities.WebSearch;
  op people is AgentCapabilities.People;
  op email is AgentCapabilities.Email<
    TFolders = [{
      folder_id: "Inbox"
    }]
  >;
}
```

---

## JSON Manifest Reference (For Direct Editing)

### 1. Schema Compliance
- **Always use schema version v1.5**: Set `"version": "v1.5"` in manifest
- **Include schema reference**: Add `"$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.5/schema.json"`
- **Validate JSON structure**: Ensure all objects follow the exact schema definitions
- **Respect property constraints**: Only use documented properties (unrecognized properties invalidate the manifest)

### 2. Required Fields Best Practices

#### Name (Required, max 100 characters)
- Use clear, descriptive names that indicate the agent's purpose
- Avoid generic terms like "Helper" or "Assistant"
- Follow naming convention: "[Function] [Context] Agent" (e.g., "Sales Data Analysis Agent")
- Support localization with `[[key_name]]` syntax when needed

#### Description (Required, max 1000 characters)
- Provide comprehensive overview of agent capabilities
- Include target use cases and expected outcomes
- Mention key data sources and integrations
- Use actionable language that sets user expectations

#### Instructions (Required, max 8000 characters)
- Write detailed behavioral guidelines covering:
  - Primary functions and responsibilities
  - Interaction patterns and tone
  - Data handling and privacy considerations
  - Response formatting preferences
  - Specific behaviors to avoid
- Use clear, imperative language
- Include examples of expected responses
- Define scope boundaries explicitly

### 3. Capabilities Configuration

#### Capability Selection Strategy
- **WebSearch**: Use for external information needs with optional site constraints (max 4 sites)
- **OneDriveAndSharePoint**: Essential for document-based agents; use specific IDs or URLs for scoping
- **GraphConnectors**: Integrate external data sources; specify connection IDs and filters
- **GraphicArt**: Enable for creative content generation scenarios
- **CodeInterpreter**: Include for data analysis, calculations, and visualization needs
- **Dataverse**: Access business data; specify exact tables and skills
- **TeamsMessages**: Search Teams communications (max 5 URLs for scoping)
- **Email**: Access mailbox data with optional folder constraints
- **People**: Essential for HR and organizational queries
- **ScenarioModels**: Use task-specific models when available
- **Meetings**: New in v1.5 - search meeting information

#### Capability Scoping Best Practices
- **Principle of least privilege**: Only grant necessary capabilities
- **Specific over general**: Use constraints (URLs, IDs, filters) to limit scope
- **Performance optimization**: Narrower scopes improve response speed
- **Security consideration**: Broader access requires additional governance

### 4. Advanced Features Implementation

#### Conversation Starters (Optional, max 6)
- Create diverse examples covering different use cases
- Use specific, actionable prompts rather than generic questions
- Include both simple and complex scenarios
- Ensure each starter demonstrates unique agent capabilities
- Format: `{"title": "Short Label", "text": "Specific question or request"}`

#### Actions/Plugins (Optional, max 10)
- Reference API plugin manifests for external integrations
- Use descriptive IDs that indicate plugin purpose
- Ensure plugin manifests follow OpenAPI specifications
- Test all actions thoroughly before deployment
- Document plugin dependencies and requirements

#### Behavior Overrides (Optional)
- **Suggestions**: Set `"disabled": true` to remove follow-up suggestions
- **Special Instructions**: Use `"discourage_model_knowledge": true` to prioritize grounded data
- Apply overrides judiciously based on specific use cases

#### Disclaimer (Optional, max 500 characters)
- Include for legal, compliance, or liability considerations
- Use clear, concise language about limitations or risks
- Required for agents handling sensitive data or providing advice

### 5. Development Workflow

#### Planning Phase
1. **Define agent purpose**: Clear problem statement and success criteria
2. **Identify data sources**: Required capabilities and access patterns
3. **Design user experience**: Conversation flow and interaction patterns
4. **Plan governance**: Security, compliance, and access controls

#### Implementation Phase
1. **Start with minimal manifest**: Core fields only
2. **Add capabilities incrementally**: Test each addition
3. **Implement conversation starters**: Based on user journey mapping
4. **Integrate actions**: External plugins and APIs
5. **Configure behavior overrides**: Fine-tune experience

#### Testing and Validation
1. **JSON schema validation**: Use official schema for validation
2. **Functional testing**: Verify all capabilities work as expected
3. **User acceptance testing**: Validate with target audience
4. **Performance testing**: Check response times and accuracy
5. **Security review**: Ensure appropriate data access controls

### 6. Localization and Accessibility

- **Localization keys**: Use `[[key_name]]` syntax for translatable strings
- **Create localization files**: Support multiple languages
- **Accessibility**: Ensure conversation starters are clear and actionable
- **Cultural sensitivity**: Consider regional differences in communication styles

### 7. Security and Governance

#### Access Controls
- **Capability scoping**: Implement principle of least privilege
- **Data classification**: Understand sensitivity of accessible data
- **User permissions**: Align with organizational access policies
- **Audit logging**: Ensure actions are traceable

#### Compliance Considerations
- **Data residency**: Understand where data is processed
- **Retention policies**: Align with organizational requirements
- **Privacy impact**: Assess personal data handling
- **Regulatory compliance**: Meet industry-specific requirements

### 8. Performance Optimization

#### Response Speed
- **Narrow capability scope**: Reduce search space
- **Specific instructions**: Clear guidance improves response quality
- **Efficient conversation starters**: Well-crafted prompts reduce iteration

#### Resource Utilization
- **Capability selection**: Only include necessary capabilities
- **Data filtering**: Use additional_search_terms for Graph Connectors
- **Plugin optimization**: Minimize external API calls

### 9. Monitoring and Maintenance

#### Success Metrics
- **User engagement**: Conversation starter usage rates
- **Response accuracy**: User satisfaction with answers
- **Performance metrics**: Response time and error rates
- **Capability utilization**: Which features are most used

#### Continuous Improvement
- **Regular reviews**: Monthly assessment of agent performance
- **User feedback integration**: Incorporate suggestions and complaints
- **Capability updates**: Leverage new schema features as available
- **Instruction refinement**: Improve based on usage patterns

### 10. Common Pitfalls to Avoid

- **Over-scoping capabilities**: Granting unnecessary access reduces performance
- **Vague instructions**: Unclear guidance leads to inconsistent responses
- **Missing validation**: Invalid JSON structure prevents deployment
- **Ignoring constraints**: Exceeding limits causes manifest rejection
- **Poor conversation starters**: Generic examples don't demonstrate value
- **Inadequate testing**: Incomplete validation leads to poor user experience
- **Security oversights**: Inappropriate access controls create risks

## Schema v1.5 Specific Features

### New Capabilities
- **Meetings capability**: Enable searching meeting information in the organization
- Enhanced email folder filtering
- Improved Dataverse integration with skill-based configuration

### Updated Constraints
- Conversation starters reduced from 12 to 6 maximum
- Enhanced validation patterns for localization
- Stricter property name validation

## Example Implementation Patterns

### Minimal Agent
```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.5/schema.json",
  "version": "v1.5",
  "name": "Document Search Agent",
  "description": "Searches company documents and provides relevant information based on user queries.",
  "instructions": "You are a document search specialist. When users ask questions, search the available SharePoint and OneDrive content to provide accurate, well-sourced answers. Always cite your sources and indicate when information might be outdated."
}
```

### Comprehensive Agent
```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.5/schema.json",
  "version": "v1.5",
  "name": "Sales Intelligence Agent",
  "description": "Comprehensive sales support agent that analyzes opportunities, generates reports, and provides data-driven insights from CRM and communication data.",
  "instructions": "You are a sales intelligence specialist. Analyze customer data from Dataverse, review recent communications, and provide actionable insights. Always maintain confidentiality and present data in clear, business-focused language.",
  "capabilities": [
    {
      "name": "Dataverse",
      "knowledge_sources": [{
        "host_name": "org.crm.dynamics.com",
        "skill": "SalesSkill_ABC123",
        "tables": [{"table_name": "opportunity"}, {"table_name": "account"}]
      }]
    },
    {"name": "TeamsMessages"},
    {"name": "Email"},
    {"name": "People"}
  ],
  "conversation_starters": [
    {"title": "Opportunity Analysis", "text": "What are my top 3 opportunities this quarter?"},
    {"title": "Customer Insights", "text": "Show me recent interactions with Contoso Corp"},
    {"title": "Pipeline Review", "text": "Generate a pipeline health report"}
  ]
}
```

Follow these guidelines consistently to create robust, secure, and effective declarative agents that provide exceptional user experiences while maintaining organizational governance and security standards.