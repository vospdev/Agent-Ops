# 04 — MCP Server

## What

Build the standalone MCP server exposing schema/mapping/validation tools,
with Integration Copilot as its MCP client — not a console-facing
pass-through.

## Why

An earlier design treated MCP as a bare RPC hop between the console and a
Python service. This project treats real, hands-on MCP experience as an
explicit learning objective, so MCP is positioned correctly: between the
agent and its tools, with Integration Copilot as the client.

## Current behavior

No MCP server exists.

## Expected behavior

A standalone MCP server runs with 5 named tools, each with a real
input/output schema. Integration Copilot discovers and invokes them as an
MCP client. Every call is authenticated and traced.

## Technical context

- Official MCP Python SDK (or FastMCP) for the server and tool schemas
- Five named tools, each with a real input/output schema (not generic
  passthrough):
  - `inspect_schema`
  - `propose_mapping`
  - `validate_mapping`
  - `detect_pii`
  - `lookup_mapping_history`
- Integration Copilot implements the MCP **client** side: discovery,
  invocation, error handling
- Every tool call traced to Langfuse Cloud
- Runs in its own Docker Compose service behind Caddy, `restart: always`
- Authenticated tool calls between Copilot and the MCP server even though
  co-located on the same droplet — treated as a real boundary, not
  implicit trust

## Acceptance criteria

- [ ] MCP server running with all 5 tools registered with real schemas
- [ ] Copilot successfully discovers and invokes at least 2 tools as an MCP client
- [ ] A malformed tool call is rejected with a clear error, not a silent failure
- [ ] Every tool call is traced in Langfuse Cloud
- [ ] Tool calls between Copilot and the MCP server are authenticated
