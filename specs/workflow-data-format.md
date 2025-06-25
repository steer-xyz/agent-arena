# Agentic Arena - Workflow Data Format Specification

## Overview
This document defines the standardized format for workflow submissions across different AI coding tools. It includes specifications for both raw export formats and the normalized format used internally by the platform.

## Universal Workflow Schema

### Standard Workflow Format (SWF)
All workflows are converted to this standardized format for consistent evaluation.

```json
{
  "version": "1.0",
  "metadata": {
    "tool_name": "claude_code",
    "tool_version": "1.5.2",
    "model_name": "claude-3-5-sonnet-20241022",
    "user_id": "user123",
    "session_id": "session_abc123",
    "task_description": "Build a React component for data visualization",
    "start_time": "2024-01-01T10:00:00Z",
    "end_time": "2024-01-01T10:05:30Z",
    "total_duration": 330,
    "success": true,
    "final_output_description": "Created Chart.jsx component with D3.js integration"
  },
  "steps": [
    {
      "step_id": "step_001",
      "type": "user_message",
      "timestamp": "2024-01-01T10:00:00Z",
      "content": "Create a React component that renders a bar chart using D3.js",
      "tokens": {
        "input": 15,
        "output": 0
      }
    },
    {
      "step_id": "step_002", 
      "type": "assistant_response",
      "timestamp": "2024-01-01T10:00:05Z",
      "content": "I'll create a React component for a bar chart using D3.js...",
      "tokens": {
        "input": 15,
        "output": 450
      },
      "tool_calls": [
        {
          "tool_name": "create_file",
          "input": {
            "path": "Chart.jsx",
            "content": "import React, { useEffect, useRef } from 'react';\nimport * as d3 from 'd3';\n\nconst Chart = ({ data }) => {\n  // Component implementation\n};\n\nexport default Chart;"
          },
          "output": {
            "success": true,
            "file_path": "Chart.jsx",
            "file_size": 1250
          }
        }
      ]
    }
  ],
  "summary": {
    "total_tokens": {
      "input": 1250,
      "output": 1800,
      "total": 3050
    },
    "tool_calls": {
      "total": 3,
      "by_type": {
        "create_file": 2,
        "edit_file": 1
      }
    },
    "files_created": 2,
    "files_modified": 1,
    "estimated_cost": 0.031,
    "iterations": 1
  },
  "validation": {
    "schema_version": "1.0",
    "checksum": "sha256:abc123...",
    "validated_at": "2024-01-01T10:10:00Z",
    "validation_errors": []
  }
}
```

## Tool-Specific Formats

### Claude Code Format
Claude Code exports workflow data in a structured JSON format.

```json
{
  "export_version": "1.0",
  "session_metadata": {
    "id": "session_abc123",
    "model": "claude-3-5-sonnet-20241022",
    "started_at": "2024-01-01T10:00:00Z",
    "ended_at": "2024-01-01T10:05:30Z",
    "total_messages": 8,
    "total_tokens": 3050
  },
  "conversation": [
    {
      "id": "msg_001",
      "role": "user",
      "timestamp": "2024-01-01T10:00:00Z",
      "content": "Create a React component for data visualization",
      "token_count": 15
    },
    {
      "id": "msg_002",
      "role": "assistant", 
      "timestamp": "2024-01-01T10:00:05Z",
      "content": "I'll help you create a React component...",
      "token_count": {
        "input": 15,
        "output": 450
      },
      "tool_uses": [
        {
          "type": "str_replace_editor",
          "parameters": {
            "command": "create",
            "path": "Chart.jsx",
            "file_text": "import React..."
          },
          "result": {
            "type": "success",
            "file_created": true,
            "file_size": 1250
          }
        }
      ]
    }
  ],
  "files": {
    "created": [
      {
        "path": "Chart.jsx",
        "size": 1250,
        "created_at": "2024-01-01T10:00:05Z"
      }
    ],
    "modified": []
  },
  "usage_summary": {
    "total_input_tokens": 1250,
    "total_output_tokens": 1800,
    "total_tokens": 3050,
    "estimated_cost": 0.031
  }
}
```

### Cursor Format
Cursor session logs with agent interactions.

```json
{
  "session_id": "cursor_session_xyz789",
  "agent_version": "0.42.0",
  "model": "gpt-4",
  "start_time": "2024-01-01T10:00:00Z",
  "end_time": "2024-01-01T10:05:30Z",
  "project_context": {
    "workspace_path": "/Users/dev/project",
    "files_in_context": ["package.json", "src/App.js"],
    "git_commit": "abc123def456"
  },
  "chat_history": [
    {
      "id": "chat_001",
      "role": "user",
      "timestamp": "2024-01-01T10:00:00Z",
      "message": "Help me create a chart component",
      "context_files": ["src/components/"]
    },
    {
      "id": "chat_002",
      "role": "assistant",
      "timestamp": "2024-01-01T10:00:03Z",
      "message": "I'll create a chart component for you...",
      "actions": [
        {
          "type": "file_create",
          "file_path": "src/components/Chart.jsx",
          "content": "import React from 'react'...",
          "success": true
        }
      ],
      "token_usage": {
        "prompt_tokens": 850,
        "completion_tokens": 420,
        "total_tokens": 1270
      }
    }
  ],
  "file_operations": [
    {
      "operation": "create",
      "file_path": "src/components/Chart.jsx",
      "timestamp": "2024-01-01T10:00:03Z",
      "size_bytes": 1250
    }
  ],
  "total_usage": {
    "prompt_tokens": 2100,
    "completion_tokens": 1200,
    "total_tokens": 3300,
    "cost_usd": 0.033
  }
}
```

### Bolt.new Format
Bolt.new exports with container and deployment information.

```json
{
  "project_id": "bolt_proj_123abc",
  "project_name": "data-viz-component",
  "created_at": "2024-01-01T10:00:00Z",
  "completed_at": "2024-01-01T10:05:30Z",
  "model": "claude-3-5-sonnet",
  "prompt_history": [
    {
      "id": "prompt_001",
      "user_input": "Create a React component for bar charts",
      "timestamp": "2024-01-01T10:00:00Z",
      "ai_response": {
        "text": "I'll create a React bar chart component...",
        "actions": [
          {
            "type": "create_file",
            "path": "src/Chart.jsx",
            "content": "import React...",
            "result": "success"
          },
          {
            "type": "update_package_json",
            "dependency": "d3",
            "version": "^7.8.5",
            "result": "success"
          }
        ],
        "tokens": {
          "input": 25,
          "output": 380
        }
      }
    }
  ],
  "file_tree": {
    "src/Chart.jsx": {
      "size": 1250,
      "created": "2024-01-01T10:00:02Z",
      "language": "jsx"
    },
    "package.json": {
      "size": 450,
      "modified": "2024-01-01T10:00:02Z",
      "language": "json"
    }
  },
  "container_info": {
    "runtime": "node:18",
    "build_time": 45,
    "deployment_url": "https://bolt-abc123.netlify.app"
  },
  "usage_stats": {
    "total_prompts": 3,
    "total_input_tokens": 450,
    "total_output_tokens": 1200,
    "total_tokens": 1650,
    "estimated_cost": 0.017
  }
}
```

### Lovable Format
Lovable AI coding assistant format.

```json
{
  "session_id": "lovable_session_456",
  "app_name": "chart-component-app",
  "start_timestamp": "2024-01-01T10:00:00Z",
  "end_timestamp": "2024-01-01T10:05:30Z",
  "model": "gpt-4-turbo",
  "interactions": [
    {
      "id": "interaction_001",
      "user_message": "I need a React component for displaying bar charts",
      "timestamp": "2024-01-01T10:00:00Z",
      "ai_response": {
        "message": "I'll create a professional bar chart component...",
        "code_changes": [
          {
            "type": "create",
            "file_path": "components/BarChart.jsx",
            "content": "import React, { useEffect, useRef } from 'react'...",
            "lines_added": 45
          },
          {
            "type": "modify",
            "file_path": "App.jsx",
            "changes": [
              {
                "line": 5,
                "old": "import './App.css';",
                "new": "import './App.css';\nimport BarChart from './components/BarChart';"
              }
            ],
            "lines_added": 1,
            "lines_removed": 0
          }
        ],
        "token_usage": {
          "input_tokens": 120,
          "output_tokens": 480,
          "total_tokens": 600
        }
      }
    }
  ],
  "project_state": {
    "files_created": 1,
    "files_modified": 1,
    "total_lines_added": 46,
    "total_lines_removed": 0,
    "dependencies_added": ["d3", "prop-types"]
  },
  "total_metrics": {
    "interactions": 3,
    "total_input_tokens": 380,
    "total_output_tokens": 1450,
    "total_tokens": 1830,
    "cost_estimate": 0.023,
    "duration_seconds": 330
  }
}
```

## Format Converters

### Claude Code to SWF Converter
```python
def convert_claude_code_to_swf(claude_export: dict) -> dict:
    """Convert Claude Code export to Standard Workflow Format"""
    swf = {
        "version": "1.0",
        "metadata": {
            "tool_name": "claude_code",
            "tool_version": claude_export.get("export_version", "unknown"),
            "model_name": claude_export["session_metadata"]["model"],
            "session_id": claude_export["session_metadata"]["id"],
            "start_time": claude_export["session_metadata"]["started_at"],
            "end_time": claude_export["session_metadata"]["ended_at"],
            "success": True  # Inferred from successful export
        },
        "steps": [],
        "summary": {
            "total_tokens": {
                "input": claude_export["usage_summary"]["total_input_tokens"],
                "output": claude_export["usage_summary"]["total_output_tokens"],
                "total": claude_export["usage_summary"]["total_tokens"]
            },
            "estimated_cost": claude_export["usage_summary"]["estimated_cost"]
        }
    }
    
    # Convert conversation to steps
    for msg in claude_export["conversation"]:
        step = {
            "step_id": msg["id"],
            "type": msg["role"] + "_message",
            "timestamp": msg["timestamp"],
            "content": msg["content"],
            "tokens": {
                "input": msg.get("token_count", {}).get("input", 0),
                "output": msg.get("token_count", {}).get("output", 0)
            }
        }
        
        # Add tool calls if present
        if "tool_uses" in msg:
            step["tool_calls"] = []
            for tool_use in msg["tool_uses"]:
                step["tool_calls"].append({
                    "tool_name": tool_use["type"],
                    "input": tool_use["parameters"],
                    "output": tool_use["result"]
                })
        
        swf["steps"].append(step)
    
    return swf
```

### Cursor to SWF Converter
```python
def convert_cursor_to_swf(cursor_export: dict) -> dict:
    """Convert Cursor session log to Standard Workflow Format"""
    swf = {
        "version": "1.0",
        "metadata": {
            "tool_name": "cursor",
            "tool_version": cursor_export["agent_version"],
            "model_name": cursor_export["model"],
            "session_id": cursor_export["session_id"],
            "start_time": cursor_export["start_time"],
            "end_time": cursor_export["end_time"],
            "success": True
        },
        "steps": [],
        "summary": {
            "total_tokens": {
                "input": cursor_export["total_usage"]["prompt_tokens"],
                "output": cursor_export["total_usage"]["completion_tokens"],
                "total": cursor_export["total_usage"]["total_tokens"]
            },
            "estimated_cost": cursor_export["total_usage"]["cost_usd"]
        }
    }
    
    # Convert chat history to steps
    for chat in cursor_export["chat_history"]:
        step = {
            "step_id": chat["id"],
            "type": chat["role"] + "_message",
            "timestamp": chat["timestamp"],
            "content": chat["message"],
            "tokens": {
                "input": chat.get("token_usage", {}).get("prompt_tokens", 0),
                "output": chat.get("token_usage", {}).get("completion_tokens", 0)
            }
        }
        
        # Add file operations as tool calls
        if "actions" in chat:
            step["tool_calls"] = []
            for action in chat["actions"]:
                step["tool_calls"].append({
                    "tool_name": action["type"],
                    "input": {
                        "file_path": action.get("file_path"),
                        "content": action.get("content")
                    },
                    "output": {"success": action.get("success", False)}
                })
        
        swf["steps"].append(step)
    
    return swf
```

## Validation Schema

### JSON Schema for Standard Workflow Format
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["version", "metadata", "steps", "summary"],
  "properties": {
    "version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+$"
    },
    "metadata": {
      "type": "object",
      "required": ["tool_name", "start_time", "end_time"],
      "properties": {
        "tool_name": {
          "type": "string",
          "enum": ["claude_code", "cursor", "bolt44", "lovable", "other"]
        },
        "tool_version": {"type": "string"},
        "model_name": {"type": "string"},
        "session_id": {"type": "string"},
        "start_time": {"type": "string", "format": "date-time"},
        "end_time": {"type": "string", "format": "date-time"},
        "total_duration": {"type": "integer", "minimum": 0},
        "success": {"type": "boolean"}
      }
    },
    "steps": {
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "object",
        "required": ["step_id", "type", "timestamp", "content"],
        "properties": {
          "step_id": {"type": "string"},
          "type": {
            "type": "string",
            "enum": ["user_message", "assistant_message", "system_message"]
          },
          "timestamp": {"type": "string", "format": "date-time"},
          "content": {"type": "string"},
          "tokens": {
            "type": "object",
            "properties": {
              "input": {"type": "integer", "minimum": 0},
              "output": {"type": "integer", "minimum": 0}
            }
          },
          "tool_calls": {
            "type": "array",
            "items": {
              "type": "object",
              "required": ["tool_name", "input", "output"],
              "properties": {
                "tool_name": {"type": "string"},
                "input": {"type": "object"},
                "output": {"type": "object"}
              }
            }
          }
        }
      }
    },
    "summary": {
      "type": "object",
      "required": ["total_tokens"],
      "properties": {
        "total_tokens": {
          "type": "object",
          "required": ["input", "output", "total"],
          "properties": {
            "input": {"type": "integer", "minimum": 0},
            "output": {"type": "integer", "minimum": 0},
            "total": {"type": "integer", "minimum": 0}
          }
        },
        "tool_calls": {
          "type": "object",
          "properties": {
            "total": {"type": "integer", "minimum": 0},
            "by_type": {"type": "object"}
          }
        },
        "estimated_cost": {"type": "number", "minimum": 0}
      }
    },
    "validation": {
      "type": "object",
      "properties": {
        "schema_version": {"type": "string"},
        "checksum": {"type": "string"},
        "validated_at": {"type": "string", "format": "date-time"},
        "validation_errors": {"type": "array"}
      }
    }
  }
}
```

## Processing Pipeline

### Workflow Ingestion Process
1. **File Upload**: User uploads workflow file
2. **Format Detection**: Automatic detection of export format
3. **Validation**: Schema validation against tool-specific format
4. **Conversion**: Transform to Standard Workflow Format
5. **Normalization**: Clean and standardize data
6. **Storage**: Store both original and normalized formats
7. **Queuing**: Add to evaluation queue

### Error Handling
```python
class WorkflowFormatError(Exception):
    """Base exception for workflow format errors"""
    pass

class UnsupportedFormatError(WorkflowFormatError):
    """Raised when file format is not supported"""
    pass

class ValidationError(WorkflowFormatError):
    """Raised when workflow data fails validation"""
    def __init__(self, message: str, errors: list):
        super().__init__(message)
        self.errors = errors

class ConversionError(WorkflowFormatError):
    """Raised when conversion to SWF fails"""
    pass
```

## Tool Integration Guidelines

### Adding New Tool Support
1. **Format Analysis**: Analyze the tool's export format
2. **Schema Definition**: Create JSON schema for validation
3. **Converter Implementation**: Write converter to SWF
4. **Test Cases**: Create comprehensive test cases
5. **Documentation**: Update this specification
6. **Parser Registration**: Register parser in the system

### Best Practices for Tool Vendors
1. **Consistent Metadata**: Include session metadata in exports
2. **Token Tracking**: Provide detailed token usage information
3. **Timestamps**: Include precise timestamps for all interactions
4. **Tool Call Logging**: Log all tool/function calls with inputs/outputs
5. **Cost Estimation**: Provide cost estimates when possible
6. **Validation**: Include checksums or validation information
7. **Versioning**: Version your export format and include version info

## Future Considerations

### Planned Enhancements
- Support for multi-file project workflows
- Video/screenshot attachment support
- Real-time workflow streaming
- Collaborative workflow support
- Advanced diff tracking for code changes
- Integration with version control systems

### Extensibility
The SWF format is designed to be extensible through:
- Additional metadata fields
- Custom tool call types
- Plugin-specific data sections
- Vendor-specific extensions in metadata
- Forward-compatible versioning scheme