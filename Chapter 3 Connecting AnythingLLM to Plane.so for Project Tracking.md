
## Chapter 3: Connecting AnythingLLM to Plane.so for Project Tracking

Integrating AnythingLLM with Plane.so for project tracking involves setting up a custom Model Context Protocol (MCP) server. This server will translate natural language requests from AnythingLLM's AI agents into structured API calls that Plane.so can understand and execute.

### 3.1 Configuring the Plane.so MCP Server for AnythingLLM Integration

The most effective strategy for connecting AnythingLLM to Plane.so is to develop a custom MCP server. This approach is preferred due to Plane.so's robust REST API and the significant advantages of using MCP for reliable LLM interaction. The custom MCP server will serve as an intermediary, encapsulating Plane.so's API endpoints and exposing them as well-defined "tools" that AnythingLLM's agents can discover and invoke.12 This method provides a standardized, robust, and LLM-friendly interface.

#### Prerequisites for Building a Custom MCP Server

Before commencing the development of a custom MCP server, several prerequisites must be met:

*   Development Stack: Select a suitable programming language and its corresponding MCP SDK. Popular and well-supported choices include Python, utilizing the anthropic-model-context-protocol SDK 30, or Node.js/TypeScript.31 The choice should align with the developer's expertise and existing infrastructure.
*   Runtime Environment: Ensure that the necessary runtime environment for the chosen language (e.g., Python, Node.js) is correctly installed on the host machine where the custom MCP server will operate.31 This ensures the server can execute its code.
*   Plane.so API Key: The Plane.so API Key, generated as detailed in Section 2.2, is essential. This key will be used by the MCP server to authenticate its requests to the Plane.so API.

#### Steps to Create a Basic Plane.so MCP Server (Conceptual Walkthrough)

The process of building a custom MCP server involves defining how it will interact with Plane.so's API and how AnythingLLM will then access these functionalities.

1.  Define Plane.so API Interactions: The initial step involves identifying the specific Plane.so API endpoints that will be exposed as callable tools for the AI agent. For project tracking, common functionalities include "Add Project," "Add Issue," "List Projects," and "Update Issue."
    *   Example: Add Project: This functionality typically involves a POST request to the Plane.so API at /api/v1/workspaces/{workspace-slug}/projects/.6 The request body must be in JSON format and include essential parameters such as
    name, identifier, and description for the new project.6
    *   Example: Add Issue: To create a new issue within a project, a POST request is sent to /api/v1/workspaces/:workspace-slug/projects/:project\_id/issues/.33 The JSON body for this request would include fields like
    name (required), description\_html or description\_stripped, priority, start\_date, target\_date, state (UUID), assignees (array of UUIDs), and labels (array of UUIDs).33
    *   Example: List Projects: Retrieving a list of all projects in a workspace is accomplished with a GET request to https://api.plane.so/api/v1/workspaces/{workspace-slug}/projects/.5 This endpoint can be further refined using
    fields and expand query parameters to retrieve only specific data points (e.g., ?fields=id,name,description) or include related resources (e.g., ?expand=members).5
    *   Example: Update Issue: Modifying an existing issue involves a PATCH request to /api/v1/workspaces/:workspace-slug/projects/:project\_id/issues/:issue\_id/.33 The request body would contain the fields to be updated, such as
    name, description, priority, or state.33
2.  Develop the Custom MCP Server: This involves writing code that implements the identified Plane.so API interactions as MCP tools. For instance, using Python with the anthropic-model-context-protocol SDK, each Plane.so API call would correspond to a function decorated as an MCP tool. This function would handle the authentication with the Plane.so API key and format the request and response appropriately. An MCP server often wraps existing APIs, adding a "conversational layer" that makes them "LLM-friendly".12 This means the server will receive a structured tool call from AnythingLLM, translate it into a Plane.so REST API request, execute it, and then return the Plane.so API's response back to AnythingLLM in an MCP-compatible format. This abstraction simplifies the interaction for the LLM, allowing it to focus on the task rather than the intricacies of the API.
3.  Configure AnythingLLM to Recognize the MCP Server: Once the custom MCP server is developed and running, AnythingLLM needs to be informed of its existence. This is achieved by editing the anythingllm\_mcp\_servers.json configuration file, located in the AnythingLLM storage plugins directory.28 This JSON file will contain an entry for the new Plane.so MCP server, specifying its command (for StdIO transport) or URL (for SSE/Streamable transport) and any arguments or headers required.28
    *   Example anythingllm\_mcp\_servers.json entry (conceptual):
    JSON
    {
    "mcpServers": {
    "plane-project-manager": {
    "command": "python3",
    "args":,
    "anythingllm": {
    "autoStart": true
    }
    }
    }
    }

    *   It is crucial to ensure that the Python executable (or Node.js, etc.) and the plane\_mcp\_server.py script are accessible from the host machine's PATH or specified with their full paths.29 AnythingLLM does not automatically install these dependencies. Setting
    autoStart: true will instruct AnythingLLM to attempt to launch this MCP server when the "Agent Skills" page is opened or an agent is invoked.29

#### Test the MCP Server Integration in AnythingLLM

After configuring AnythingLLM, navigate to the "Settings -> Agent Skills" page in the AnythingLLM UI. The newly added Plane.so MCP server should be visible, and its status can be monitored.28 From here, the server can be started, stopped, or reloaded if configuration changes are made.29

#### Craft AnythingLLM Agent Prompts to Utilize Plane.so Tools

With the MCP server integrated, AnythingLLM's AI agents can now be instructed to interact with Plane.so. This involves crafting prompts that explicitly direct the agent to use the newly exposed tools.

*   Prompting Strategy: When using AI agents in AnythingLLM, the @agent directive is used to initiate an agent session.19 The prompt should clearly state the desired action and provide all necessary parameters for the Plane.so API call. The LLM instruction block within agent flows is highly flexible and can leverage variables to dynamically shape the LLM's output.22
*   Example Agent Prompts:
    *   To create a new project: @agent Create a new project in Plane.so called "Website Redesign" with the identifier "WEBREDESIGN" and a description "Revamp the company website for modern aesthetics and improved user experience."
    *   To add an issue to an existing project: @agent Add an issue to the "Website Redesign" project (WEBREDESIGN) titled "Design Homepage Mockups" with high priority and assign it to John Doe. The description should be: "Create initial mockups for the new website homepage, focusing on visual appeal and user flow." (Assuming the MCP server can resolve "John Doe" to a Plane.so user UUID).
    *   To list projects: @agent List all projects in Plane.so.
    *   To update an issue: @agent Update the issue "Design Homepage Mockups" in project "WEBREDESIGN" to 'Completed'.

#### Process Responses from Plane.so

When an agent successfully executes a Plane.so tool via the MCP server, the response from the Plane.so API is returned to AnythingLLM. The agent can then process this response, summarize it, or use it to inform subsequent actions or provide feedback to the user. For instance, after creating a project, the agent could confirm its creation and provide the new project's identifier.

