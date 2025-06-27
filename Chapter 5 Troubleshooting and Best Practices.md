
## Chapter 5: Troubleshooting and Best Practices

Maintaining a complex integrated system requires proactive troubleshooting and adherence to best practices to ensure optimal performance and reliability.

### 5.1 Common Issues & Solutions

Even with careful setup, issues can arise. Understanding common problems and their resolutions is key to maintaining system uptime.

#### AnythingLLM Troubleshooting

*   Agent Not Working/Stuck in Loop: This can be a frustrating issue where the AI agent fails to execute tasks or enters repetitive cycles.18
    *   Solutions:
        *   Check Logs: AnythingLLM generates logs that often contain error messages or clues. These logs are typically found in the logs folder within the AnythingLLM storage directory.40
        *   Model Selection: The choice of LLM significantly impacts an agent's tool-calling ability. Some models, especially distilled versions or those not specifically optimized for tool calling (e.g., certain Deepseek R1 variants), may struggle with generating correct JSON for tool calls.18 Experimenting with smaller, more tool-call-optimized models (e.g., Llama 3.2) can resolve this.18 AnythingLLM allows different models for chat and agent functions for this reason.18
        *   Resource Limitations: Large models can be demanding on hardware, especially VRAM. Insufficient VRAM can lead to slow inference, offloading computations to slower system RAM, and agent instability or loops.18 Monitoring GPU VRAM and system RAM usage is important.
        *   Prompt Quality: The clarity and specificity of instructions provided to the LLM are paramount. If the LLM struggles to follow instructions, refining the prompt or trying a different model may be necessary.22
        *   Agent Configuration: Review the agent's steps and logic within AnythingLLM to ensure no logical errors could lead to infinite loops.18

#### Plane.so Troubleshooting

*   API Key Issues:
    *   Missing/Invalid API Key: Requests will fail with 401 Unauthorized errors if the X-API-Key header is missing or the key is invalid/expired.5
    *   Solution: Ensure the API key is correctly included in the header and is valid. Regenerate the key if compromise is suspected.5
*   Rate Limiting: Exceeding 60 requests per minute per API key will result in 429 Throttling Errors.5
    *   Solution: Implement exponential backoff or introduce delays in AI agent workflows that make frequent Plane.so API calls. Monitor X-RateLimit-Remaining and X-RateLimit-Reset headers in API responses.5
*   Docker/Self-Hosted Issues:
    *   Docker Compose Errors: Often due to insufficient permissions (lack of sudo/root privileges) or using an outdated docker-compose version.42
    *   Solution: Ensure the user has sudo or root access. Install the latest docker compose version.42
    *   Migrator Container Exited: This typically occurs if an external database is configured to run on localhost when the connection is attempted from inside a Docker container.42
    *   Solution: Host the database on a network-accessible server and update the database URL accordingly.42
    *   "Plane didn't start up" on Website: Can occur even if Docker logs are clean. A common cause for self-hosted instances is misconfiguration of WEB\_URL or CORS\_ALLOWED\_ORIGINS environment variables, or attempting to run Plane across multiple domains without custom image builds.26
    *   Solution: Ensure WEB\_URL and CORS\_ALLOWED\_ORIGINS are set to the correct FQDN (Fully Qualified Domain Name) including the NGINX\_PORT if custom.26 All Plane services must operate under the same domain unless custom images are built.43
    *   Viewing Logs: For detailed troubleshooting, access Plane.so container logs using sudo prime-cli monitor or by running ./setup.sh and selecting option 6 (View Logs) for Community Edition.44

#### MCP Server Not Starting/Connecting

*   Command Availability: The executable command for the MCP server (e.g., python3, npx) must be installed on the host machine and available in its PATH.29 AnythingLLM does not install these dependencies.
*   Configuration File Location: Ensure the anythingllm\_mcp\_servers.json file is in the correct plugins/anythingllm\_mcp\_servers.json path within the AnythingLLM storage directory.29 This file is automatically created if it doesn't exist when accessing the "Agent Skills" page.29
*   Autostart Prevention: Check if anythingllm.autoStart: false is set in the MCP server's configuration, as this prevents automatic startup.28
*   Transport Type: Verify that the type (StdIO, SSE, Streamable) and url/command fields in the MCP configuration match the server's implementation.28

### 5.2 Optimizing AI Performance and Reliability

To maximize the effectiveness and stability of the AI-powered system, several optimization strategies should be employed.

*   Prompt Engineering: The quality of prompts directly influences LLM output.
    *   Clarity and Specificity: Provide clear, detailed instructions to the LLM.22 The more descriptive the prompt, the better the output.
    *   Role Assignment: Assign a specific role to the LLM (e.g., "You are an expert project manager") to prime it with relevant knowledge.35
    *   Step-by-Step Thinking: Instruct the LLM to "take a step back and think step by step" to encourage more structured and accurate responses.35
    *   Output Format: Explicitly define the desired output format, especially for structured data like Markdown or JSON.35 Providing examples of desired output can be highly effective.35
    *   System Prompt Variables: Utilize AnythingLLM's system prompt variables (e.g., {date}, {user.name}) and custom static variables to inject dynamic and static context, making prompts more personalized and effective.21
    *   Conciseness: While detailed, avoid overly verbose instructions that might confuse the LLM.35
    *   Parameter Tuning: Experiment with LLM parameters like temperature (creativity), top-p (token diversity), max\_tokens (output length), and repetition penalty to fine-tune output quality for specific tasks.45
*   Model Selection: The choice of LLM impacts performance, cost, and tool-calling ability.
    *   Tool-Calling Capability: Select models known for strong tool-calling capabilities, as some models may struggle with generating accurate JSON for tool invocations.18
    *   Resource Alignment: Choose models that align with available hardware resources, especially GPU VRAM, to prevent performance bottlenecks.3 Consider hosted solutions for state-of-the-art responses with minimal local overhead.3
*   Resource Monitoring: Continuously monitor system resources (CPU, RAM, GPU VRAM) to identify bottlenecks. This is particularly important for local LLM deployments.3 Tools like
docker stats can provide real-time container resource usage.

### 5.3 Long-Term Maintenance and Evolution

For the sustained success and adaptability of the integrated system, ongoing maintenance and strategic evolution are crucial.

*   Regular Software Updates: Keep AnythingLLM, Plane.so, and their respective Docker images updated to the latest stable versions. This ensures access to new features, performance improvements, and security patches.16
*   API Key Rotation: Regularly rotate Plane.so API keys, especially if there's any suspicion of compromise, to maintain security.5
*   Documentation Review and Refinement: Periodically review the user manual on GitHub for accuracy, completeness, and clarity. As the integrated system evolves, the documentation must reflect these changes. This includes reviewing the effectiveness of AI-generated content and refining prompts as needed.
*   MCP Server Maintenance: Monitor the custom Plane.so MCP server for logs and errors.28 Update the MCP server code as Plane.so API changes or new functionalities are desired.
*   Backup Strategy: Implement a robust backup strategy for AnythingLLM's persistent storage and any critical Plane.so data (if self-hosted) to prevent data loss.
