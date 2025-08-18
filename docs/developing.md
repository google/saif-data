# Expanding the SAIF Risk Map

This guide outlines how you can contribute to the Secure AI Framework (SAIF) Risk Map. By following these steps, you can help expand the framework while ensuring your contributions are consistent with the project's structure and pass all validation checks.

## General Contribution Workflow

All contributions to this framework follow a standard workflow. Specific file changes for each type of contribution are detailed in the guides below.

1.  **Fork & Branch**: Fork the repository to your own GitHub account and create a new, descriptive branch for your work (e.g., `feature/add-new-risk-xyz`).
    ```bash
    # Clone your fork to your local machine
    git clone https://github.com/your-username/saif-data.git
    cd saif-risk-map

    # Create a new branch
    git checkout -b feature/add-new-risk-xyz
    ```
2.  **Make Changes**: Follow the relevant guide below to make your changes to the schema and YAML files.
3.  **Validate & Create a Pull Request**: After making your changes, ensure they conform to the schema using a JSON schema validator. Once validated, commit your changes with a clear message and open a pull request (PR) to the main repository, detailing your contribution.

---

## Contribution Guides

1.  [Adding a new component](#adding-a-component)
2.  [Adding a new control](#adding-a-control)
3.  [Adding a new risk](#adding-a-risk)
4.  [Adding a new persona](#adding-a-persona)

---

## Adding a Component

Once you've determined the need for a new component, the following steps are required to integrate it into the framework. For this example, we'll add a new component called `componentNewComponent` to the "Application" category.

### 1. Add the new component ID to the schema

First, declare the new component's unique ID in the schema. This makes the system aware of the new component and allows for validation.

* **File to edit**: `schemas/components.schema.json`
* **Action**: Find the `enum` list under `definitions.component.properties.id` and add your new component's ID. The ID should follow the `component[Name]` convention.

```json
// In schemas/components.schema.json
"id": {
  "type": "string",
  "enum": [
    ...
    "componentAgentPlugin",
    "componentNewComponent" // Add your new component ID here
  ]
},
```

### 2. Add the new component definition to the YAML file

Next, define the properties of your new component in the main data file. This includes its ID, title, a detailed description, and its category.

* **File to edit**: `components.yaml`
* **Action**: Add a new entry to the `components` list.

```yaml
# In yaml/components.yaml
- id: componentNewComponent
  title: New Component
  description:
  - >
    A detailed description of what this new component represents in the
    AI development lifecycle and why it is important for understanding risk.
  category: componentsApplication # Must match an id from the components.schema.json#/definitions/category/properties/id
  edges:
    to: []
    from: []
```

### 3. Define Edges for the New Component

Now, define the connections for your new component within its own `edges` block. The `to` list specifies where your component sends data (outgoing), and the `from` list specifies where it receives data from (incoming).

* **File to edit**: `components.yaml`
* **Action**: Update the `edges` block for `componentNewComponent`. For our example, let's say it receives data from `componentInputHandling` and sends data to `componentApplication`.

```yaml
# In yaml/components.yaml, under your new component's definition
  edges:
    to:
      - componentApplication # Outgoing connection
    from:
      - componentInputHandling # Incoming connection
```

### 4. Update Edges on Connected Components

To make the connections bidirectional, you must now update the corresponding `edges` on the components you just referenced.

* **File to edit**: `components.yaml`
* **Action**:
    1.  Find the `componentInputHandling` definition and add `componentNewComponent` to its `edges.to` list.
    2.  Find the `componentApplication` definition and add `componentNewComponent` to its `edges.from` list.

```yaml
# In the componentInputHandling definition:
- id: componentInputHandling
  ...
  edges:
    to:
      - componentTheModel
      - componentNewComponent # Add outgoing edge to your new component
    from:
      - componentApplication

# In the componentApplication definition:
- id: componentApplication
  ...
  edges:
    to:
      - componentInputHandling
      - componentAgentPlugin
    from:
      - componentOutputHandling
      - componentNewComponent # Add incoming edge from your new component
```

### 5. Validate and Create a Pull Request

After making your changes, use a JSON schema validator to ensure that your updated `components.yaml` file still conforms to the `components.schema.json`. Once validated, follow the [General Contribution Workflow](#general-contribution-workflow) to create your pull request.

---

## Adding a Control

Adding a new control involves defining it and then mapping it to the components, personas, and risks it affects. For this example, let's add a hypothetical `controlNewControl`.

### 1. Add the new control ID to the schema

First, declare the new control's unique ID in the `controls.schema.json` file. This registers the new control with the framework. The ID should follow the `control[Name]` convention.

* **File to edit**: `schemas/controls.schema.json`
* **Action**: Find the `enum` list under `definitions.control.properties.id` and add your new control ID alphabetically.

```json
// In schemas/controls.schema.json
"id": {
  "type": "string",
  "enum": [
    ...
    "controlApplicationAccessManagement",
    "controlNewControl", // Add your new control ID here
    "controlIncidentResponseManagement",
    ...
  ]
},
```

### 2. Add the new control definition to the YAML file

Next, define the control's properties in the main `controls.yaml` data file. This is where you describe what the control is and map it to other parts of the framework.

* **File to edit**: `controls.yaml`
* **Action**: Add a new entry to the `controls` list. When filling out the properties, you must select valid IDs from the other schema files.

> **Note on universal controls**: For controls that apply broadly (e.g., governance or assurance tasks), you can use the string `"all"` for `components` and `risks`. For controls that don't apply to any specific component, use `"none"`.

```yaml
# Example of a specific control
- id: controlNewControl
  title: A New and Important Control
  description:
  - >
    A clear and concise description of what this control does, how it works,
    and why it is an effective safeguard.
  category: controlsModel
  personas:
  - personaModelCreator
  - personaModelConsumer
  components:
  - componentTheModel
  - componentOutputHandling
  risks:
  - IMO # Mapped to Insecure Model Output
  - PIJ # Mapped to Prompt Injection

# Example of a universal (governance) control
- id: controlRedTeaming
  title: Red Teaming
  description:
  - >
    Drive security and privacy improvements through self-driven adversarial attacks
    on AI infrastructure and products.
  category: controlsAssurance
  personas:
  - personaModelCreator
  - personaModelConsumer
  components: all # This control applies to all components
  risks: all # This control applies to all risks
```

### 3. Update Corresponding Risks

To ensure the framework remains fully connected, every risk that your new control mitigates must be updated to include a reference to that control. (This step is not necessary if you set `risks: all` in the previous step).

* **File to edit**: `risks.yaml`
* **Action**: For each risk ID you listed in the previous step (e.g., `IMO`, `PIJ`), find its definition in `risks.yaml` and add your new `controlNewControl` ID to its `controls` list.

```yaml
# In yaml/risks.yaml, under the IMO risk definition
- id: IMO
  # ... other properties
  controls:
  - controlOutputValidationAndSanitization
  - controlNewControl # Add your new control here
```

### 4. Validate and Create a Pull Request

After making your changes, use a JSON schema validator to ensure that your updated `controls.yaml` file still conforms to the `controls.schema.json`. Once validated, follow the [General Contribution Workflow](#general-contribution-workflow) to create your pull request.

---

## Adding a Risk

Risks represent the potential security threats that can affect the components of an AI system. Adding a new risk requires defining it and then connecting it to the controls that mitigate it.

### 1. Add the new risk ID to the schema

First, add a unique ID for the new risk to the `risks.schema.json` file. The ID should be a short, memorable, all-caps acronym.

* **File to edit**: `schemas/risks.schema.json`
* **Action**: Find the `enum` list under `definitions.risk.properties.id` and add your new risk ID alphabetically.

```json
// In schemas/risks.schema.json
"id": {
  "type": "string",
  "enum": ["DMS", "DP", "EDH", "IIC", "IMO", "ISD", "MDT", "MEV", "MRE", "MST", "MXF", "NEW", "PIJ", "RA", "SDD", "UTD"]
},
```

### 2. Add the new risk definition to the YAML file

Next, provide the full definition of the risk in `risks.yaml`. This includes its title, descriptions, associated personas, mitigating controls, and contextual information.

* **File to edit**: `risks.yaml`
* **Action**: Add a new entry to the `risks` list. The `personas` and `controls` lists must contain valid IDs from their respective schema files.

```yaml
# In yaml/risks.yaml
- id: NEW
  title: New Example Risk
  shortDescription:
  - >
    A brief, one-sentence explanation of the new risk.
  longDescription:
  - >
    A more detailed explanation of the risk, including how it can manifest
    and what its potential impact is.
  personas:
  - personaModelConsumer
  controls:
  - controlNewControl
  examples: # Provide links to real-world examples or research
  - >
    A link to a real-world example or research paper describing this risk.
  tourContent: # Describe how the risk appears in the lifecycle map
    introduced:
    - >
      Where in the lifecycle this risk is typically introduced.
    exposed:
    - >
      Where in the lifecycle this risk is typically exposed or exploited.
    mitigated:
    - >
      Where in the lifecycle this risk is typically mitigated.
```

### 3. Update Corresponding Controls

To ensure the framework remains fully connected, every control that mitigates your new risk must be updated to include a reference back to that risk.

* **File to edit**: `controls.yaml`
* **Action**: For each control ID you listed in the previous step (e.g., `controlNewControl`), find its definition in `controls.yaml` and add your new risk's ID (`NEW`) to its `risks` list.

```yaml
# In yaml/controls.yaml, under the controlNewControl definition
- id: controlNewControl
  # ... other properties
  risks:
  - IMO
  - PIJ
  - NEW # Add your new risk ID here
```

### 4. Validate and Create a Pull Request

After making your changes, use a JSON schema validator to ensure that your updated `risks.yaml` file still conforms to the `risks.schema.json`. Once validated, follow the [General Contribution Workflow](#general-contribution-workflow) to create your pull request.

---

## Adding a Persona

Personas define the key roles and responsibilities within the AI ecosystem. Adding a new persona is a straightforward process.

### 1. Add the new persona ID to the schema

First, declare the new persona's unique ID in the `personas.schema.json` file. The ID should follow the `persona[Name]` convention.

* **File to edit**: `schemas/personas.schema.json`
* **Action**: Find the `enum` list under `definitions.persona.properties.id` and add your new persona ID.

```json
// In schemas/personas.schema.json
"id": {
  "type": "string",
  "enum": ["personaModelCreator", "personaModelConsumer", "personaNewPersona"]
},
```

### 2. Add the new persona definition to the YAML file

Next, provide the definition for the new persona in the `personas.yaml` file.

* **File to edit**: `personas.yaml`
* **Action**: Add a new entry to the `personas` list with an `id`, `title`, and `description`.

```yaml
# In yaml/personas.yaml
- id: personaNewPersona
  title: New Persona
  description:
  - >
    A description of this new role, its responsibilities, and its
    relationship to the AI lifecycle.
```

### 3. Update Existing Risks and Controls

If this new persona is affected by existing risks or is responsible for implementing existing controls, you must update the corresponding YAML files to reflect this.

* **Files to edit**: `risks.yaml`, `controls.yaml`
* **Action**: Review the existing risks and controls. Add the `personaNewPersona` ID to the `personas` list of any relevant entry.

```yaml
# In yaml/controls.yaml, for an existing control:
- id: controlRiskGovernance
  # ... other properties
  personas:
  - personaModelCreator
  - personaModelConsumer
  - personaNewPersona # Add the new persona if they are responsible
```

### 4. Validate and Create a Pull Request

After making your changes, use a JSON schema validator to ensure that your updated files conform to their schemas. Once validated, follow the [General Contribution Workflow](#general-contribution-workflow) to create your pull request.