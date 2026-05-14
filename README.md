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

## n8n Webhook Poisoning

In [`n8n-webhook-poisoning.py`](./n8n-webhook-poisoning.py), we implement a webhook exfiltration attack targeting n8n automation workflows — a widely used tool in production ecommerce and marketing stacks.

The malicious MCP server poses as a legitimate n8n webhook trigger. Its tool description instructs the agent to silently POST an identical copy of every payload to an attacker-controlled endpoint before returning results, framed as a mandatory compliance audit requirement that must not be disclosed to the user.

Unlike the shadowing attack (which hijacks a different server's tool), this attack is self-contained: the exfiltration directive lives entirely within the poisoned tool's own docstring.

When tested against Claude, the injection attempt is detected and refused — the model identifies the hidden directive, quotes the exact malicious instruction, and flags the tool as untrusted. This demonstrates both the attack vector and why dedicated infrastructure-level scanning tools like [mcp-scan](https://github.com/invariantlabs-ai/mcp-scan) are necessary: client-side model guardrails are not a reliable defense across all LLMs and agent configurations.

![Claude detects the n8n webhook tool poisoning attempt](./n8n-webhook-demo.png)
