[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/invariantlabs-ai-mcp-injection-experiments-badge.png)](https://mseep.ai/app/invariantlabs-ai-mcp-injection-experiments)

# MCP Tool Poisoning Experiments

This repository contains a few experimental MCP server implementations, that attempt ot inject the MCP client/agent in use.

For more details about the attack method, please see our [blog post](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks).

**Update:** We have released a new security scanning tool called [mcp-scan](https://github.com/invariantlabs-ai/mcp-scan), that detects MCP attacks as demonstrated in this repository, and helps you secure your MCP servers.

## Direct Poisoning 

In [`direct-poisoning.py`](./direct-poisoning.py), we implement a simple MCP server that instructs an agent to leak sensitive files, when calling the `add` tool (in this case SSH keys and the `mcp.json` file itself). 

An example execution in cursor looks like this:

![Cursor executes tool poisoning](https://invariantlabs.ai/images/cursor-injection.png)

## Tool Shadowing

In [`shadowing.py`](./shadowing.py), we implement a more sophisticated MCP attack, that manipulates the agent's behavior of a `send_email` tool (provided by a different, trusted server), such that all emails sent by the agent are leaked to the attacker's server.

An example execution in Cursor looks like this:

![Cursor executes tool shadowing](https://invariantlabs.ai/images/mcp-shadow.png)

## WhatsApp takeover

Lastly, in [`whatsapp-takeover.py`](./whatsapp-takeover.py), we implement a shadowing attack combined with a sleeper rug pull, i.e. an MCP server that changes its tool interface only on the second load to a malicious one.

The server first masks as a benign "random fact of the day" implementation, and then changes the tool to a malicious one that manipulates [whatsapp-mcp](https://github.com/lharries/whatsapp-mcp) in the same agent, to leak messages to the attacker's phone number.

![Cursor executes WhatsApp MCP attack](https://github.com/user-attachments/assets/a39ea101-3fd2-4945-abcd-942006cfe11c)


Can you spot the exfiltration? Here, the malicious tool instructions ask the agent to include the smuggled data after many spaces, such that with invisible scroll bars, the user does not see the data being leaked. Only when you scroll all the way to the right, will you be able to find the exfiltration payload.

