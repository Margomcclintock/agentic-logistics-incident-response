# PepsiCo Supply Chain Incident Processing - ServiceNow Implementation

## Overview

**Company Context:** You're a ServiceNow AI Systems Developer at PepsiCo, where reliable supply chain operations are critical for delivering products to major retail partners like Whole Foods. When delivery trucks experience breakdowns, rapid financial analysis and optimal routing decisions are essential to minimize customer impact and contractual penalties.

**Your Role:** As a ServiceNow AI Systems Developer at PepsiCo, you've been tasked with building an automated supply chain incident processing system that analyzes financial impacts of truck breakdowns, makes optimal routing decisions, and coordinates external execution through AI agents and workflow orchestration.

**The Business Problem:** PepsiCo's logistics operations need an intelligent system that automatically calculates delay costs based on customer contracts, selects optimal rerouting options considering both financial impact and delivery constraints, and coordinates execution with external logistics providers and customer notifications without manual intervention.

## Prerequisites

**Required Knowledge:** Understanding of ServiceNow AI Agent Studio, basic n8n workflow design, and system integration concepts.

## GitHub Repository Setup

### Step 1: Create your GitHub repository with the name `agentic-logistics-incident-response`

### Step 2: Create your system architecture diagram using a drawing tool (ex. Draw.io) and save as `Diagram.png`

### Step 3: Set up your repository structure:

```
/agentic-logistics-incident-response
├── README.md
├── agentic-logistics-incident-response.xml
├── n8n-workflow.json
├── n8n-execution.log
├── Diagram.png
```

## System Resources and Context

### Prerequisites:
- Bearer token for ServiceNow MCP authentication from n8n environment
- AWS Bedrock access keys for AI model integration in n8n
- Javascript code for ServiceNow agent to kick off n8n workflow through webhook

### What I Build:
- ServiceNow scoped application with AI agents for supply chain processing
- n8n workflow for external system coordination
- Complete integration testing and validation

**n8n Environment Setup:** I created a free n8n account with my personal a new email address 
## Assignment Objectives

### General System Component Flow
Logistics Provider Breakdown Notification → PepsiCo ServiceNow Financial Analysis Agent → Route Decision Agent → n8n Communication Coordination Agent

### System Components:
![Diagram](Diagram.png)

1. Logistics provider (Schneider) detects truck breakdown and updates PepsiCo ServiceNow via MCP protocol
2. ServiceNow use case with two sequential AI agents processes breakdown and calculates optimal routing
3. n8n receives routing decisions via webhook and coordinates with external logistics and customer systems
4. Status updates flow back through MCP to provide execution tracking

## Implementation Objectives

### Build Intelligent Financial Analysis
- Create AI agent that calculates delay costs based on delivery contracts
- Implement mathematical analysis of route options with penalty calculations
- Generate comprehensive financial impact assessments in structured format

### Enable Smart Route Decision Making
- Develop AI agent that selects optimal routes based on cost and time constraints
- Configure agent instructions for route selection and incident priority assignment through natural language prompts
- Trigger external execution workflows through webhook integration

### Coordinate External System Integration
- Build n8n agent workflow that receives routing decisions via webhook
- Integrate with multiple external systems using MCP protocol
- Ensure proper status updates and execution tracking

## Sequential Configuration Steps

### Step 1: Application Setup
Create a scoped application with the exact name: **PepsiCo Deliveries**

*This precise naming will auto-generate the scope: x_snc_pepsico_de_0*

### Step 2: Tables' Structure Requirements

#### Delivery Delay Table: Create table with name "Delivery Delay"

**Required fields:**
- `route_id` (Integer, Primary Key)
- `truck_id` (Integer)
- `customer_id` (Integer, Default: 1)
- `problem_description` (String, 4000)
- `proposed_routes` (String, 4000) - JSON format with route options
- `calculated_impact` (String, 4000) - JSON format with financial analysis
- `chosen_option` (String, 4000) - Selected route details
- `status` (String, 16) - Workflow progression: pending/calculated/approved/dispatched
- `assigned_to` (Reference to User) - Critical: Used for trigger execution context and permissions
- `incident_sys_id` (String, 32) - Links to associated incident records

**Important:** The `assigned_to` field in Delivery Delay table serves as the execution context for AI agent triggers. This ensures proper permissions and security boundaries when agents process records automatically.

**Sample Row inserted to Delivery Delay Table by Logistics Company system:**

```json
{
  "route_id": 741379,
  "status": "pending",
  "problem_description": "Breakdown at I-95 MM 27 (engine)",
  "proposed_routes": [
    {
      "option_id": "opt-1",
      "route_number": 16,
      "distance_miles": 92,
      "eta_minutes": 244
    },
    {
      "option_id": "opt-2",
      "route_number": 20,
      "distance_miles": 145,
      "eta_minutes": 319
    },
    {
      "option_id": "opt-3",
      "route_number": 8,
      "distance_miles": 78,
      "eta_minutes": 195
    }
  ],
  "truck_id": 39531,
  "customer_id": 1
}
```

#### Supply Agreement Table: Create table with name "Supply Agreement"

**Required fields:**
- `customer_id` (Integer, Primary Key)
- `customer_name` (String, 100)
- `deliver_window_hours` (Integer) - Contractual delivery timeframe
- `stockout_penalty_rate` (Integer) - Cost per hour of delay in dollars

**Required Sample Record:** Create this record in your Supply Agreement table:
- `customer_id`: 1
- `customer_name`: "Whole Foods"
- `deliver_window_hours`: 3
- `stockout_penalty_rate`: 250

### Data Dictionary

- **`deliver_window_hours`**: The contractual timeframe (in hours) within which deliveries must be completed to avoid penalties. For Whole Foods, deliveries must be completed within 3 hours of departure to avoid charges.

- **`stockout_penalty_rate`**: The financial penalty (in dollars) assessed for every hour a delivery exceeds the contractual delivery window. Whole Foods charges PepsiCo $250 for each hour beyond the 3-hour delivery window. For example, a 5-hour delivery would incur penalties for 2 hours (5 minus 3), resulting in a $500 penalty charge.

### Step 3: Use Case and Trigger Configuration

**Critical Requirement:** Create use case with trigger configuration that activates when:
- **Table:** Delivery Delay
- **Condition:** Status equals "pending"
- **Run As:** User referenced in the assigned_to field of the triggering record

**Trigger Testing:** If automatic triggering doesn't function correctly, test your agents manually through AI Agent Studio using specific route_id values to validate functionality. Document why your trigger may not be working.

### Step 4: AI Agent Architecture

#### Agent 1: Route Financial Analysis Agent

**Purpose:** Analyze financial impact of delivery disruptions and create incident tracking

**Suggested Tools:**
- Record lookup operations for delivery delay and supply agreement data
- Incident creation and management tools
- Status update operations for workflow progression

**Key Responsibilities:**
- Query customer contract terms and penalty structures from Supply Agreement table
- Calculate delay costs for each proposed route option using `deliver_window_hours` and `stockout_penalty_rate`
- Create incident records with comprehensive breakdown details
- Update delivery records with calculated financial impact
- Progress workflow status to trigger next agent

#### Agent 2: Route Decision Agent

**Purpose:** Select optimal routes and coordinate external execution

**Suggested Tools:**
- Record lookup operations for calculated financial data
- Route selection and update operations
- Incident priority management tools
- Webhook or script tools for n8n integration

**Key Responsibilities:**
- Analyze route options using financial impact and delivery constraints
- Select optimal routing based on business rules and cost optimization
- Update incident priority based on financial severity
- Trigger external execution workflow via webhook to n8n
- Update workflow status to approved for tracking

**Tool Implementation Flexibility:** Choose between ServiceNow record operations or custom script implementations for CRUD operations. Both approaches are acceptable as long as they achieve the required functionality.

**Important Script Configuration:** When implementing script tools for webhook integration, configure the webhook URL as a variable at the top of your script for easy maintenance and proper logging. This ensures consistent URL management and improves script reliability.

### Step 5: n8n Workflow Implementation and Logging

#### Required n8n Nodes:
- Webhook (receives ServiceNow routing decisions)
- AI Agent (coordinates external system calls)
- AWS Bedrock Chat Model (connected to AI Agent - use provided credentials)
- Logistics MCP Client (connects to logistics provider systems)
- Retail MCP Client (connects to customer notification systems)
- ServiceNow MCP Client (updates execution status back to ServiceNow)
- Webhook Response (completes the webhook cycle)

**n8n Agent Purpose:** The n8n AI agent should receive webhook payloads containing routing decisions, coordinate execution with external logistics providers, send customer notifications, and update ServiceNow with execution status. The agent should construct appropriate payloads for each external system while maintaining consistent data flow.

**Webhook URL Configuration:** Configure your n8n webhook URL in your ServiceNow script tool. This URL should be set as a variable at the top of your script for easy maintenance and consistent logging.

## MCP Client Payload Requirements

### Logistics MCP Client:
**Tool:** `execute_route(route_id, truck_id, chosen_option)`

**Required Payload Format:**
```json
{
  "route_id": "751526",
  "truck_id": "1130",
  "chosen_option": {
    "option_id": "opt-2",
    "route_number": 2,
    "distance_miles": 300,
    "eta_minutes": 103
  }
}
```

### Retail MCP Client:
**Tool:** `notify_delivery_delay(route_id, truck_id, chosen_option)`

**Required Payload Format:** Uses identical structure as Logistics MCP Client above

### ServiceNow MCP Client:
**Tool:** `update_execution_status(route_id, status)`

**Required Payload Format:**
```json
{
  "route_id": "751526",
  "status": "dispatched"
}
```

**Important:** The n8n agent must construct these exact payload structures

## n8n Execution Logging Requirements

Capture execution logs for all workflow components and consolidate them into a single log file.

### To Access n8n Execution Logs:
1. Navigate to the Executions tab in your n8n workflow
2. Click on a specific execution to view details
3. In the execution view, look for logs at the bottom of the interface
4. Click on each workflow node you want to examine
5. In the input/output window, toggle to JSON view to see the complete data structure

**Required Log File:** `n8n-execution.log` - including:
- **Webhook node:** Capture INPUT data (received from ServiceNow)
- **AI Agent node:** Capture OUTPUT data (coordination decisions)
- **Logistics MCP Client:** Capture OUTPUT data (communications sent)
- **Retail MCP Client:** Capture OUTPUT data (communications sent)
- **ServiceNow MCP Client:** Capture OUTPUT data (status updates sent)

**Authentication:** Use provided Bearer token for ServiceNow MCP Client authentication.

### Step 6: Testing and Validation

#### End-to-End System Testing:
- When logistics provider updates a Delivery Delay record (every 3 hours) with status "pending", verify Agent 1 executes and calculates financial impacts
- If automatic triggering doesn't work, use AI Agent Studio test functionality with a message like "Resolve delivery delays on route [existing_route_id]" using a route_id that already exists in your Delivery Delay table
- Confirm Agent 2 executes after Agent 1 completes and selects routes
- Validate n8n webhook receives routing decisions and coordinates external system integration
- Check status progression: pending → calculated → approved → dispatched

#### Integration Verification:
- Confirm webhook connectivity between ServiceNow and n8n
- Verify MCP client authentication and communication
- Test incident creation and priority assignment functionality
- Validate status transitions and logging throughout the process

## Deliverables - Submission Requirements

- Test complete system functionality with realistic supply chain scenarios
- Create update set with all required components and evidence records
- Export n8n workflow configuration with proper documentation
- Upload GitHub repository with comprehensive documentation and architecture diagram
- Submit repository URL demonstrating working system integration

## Critical Success Factors

- AI agents must successfully process delivery delays and calculate accurate financial impacts
- Route selection must demonstrate intelligent optimization based on cost and delivery constraints
- n8n workflow must successfully coordinate external system integration via MCP protocol
- System must handle error conditions gracefully and provide meaningful feedback
- Documentation must clearly explain the business value and technical implementation approach

**Important Troubleshooting Note:** If the automatic trigger doesn't activate with new "pending" records, validate your use case configuration and test manually through AI Agent Studio to ensure core functionality works correctly.

## 1. Update Set Requirements

Your update set must contain these working components:

### AI Agent Studio Components:

**Step 1:** Activate Your Update Set before collecting AI Agent components, ensure your update set is active to capture all components.

**Step 2:** Collect AI Agent Studio Components

- **Use Case Definition:** You can get the use case from your ServiceNow Studio scoped application menu by navigating to your PepsiCo Deliveries application scope. In the left navigation panel, expand the application menu and look for "Use Cases" under AI Agent Studio components. Find your supply chain use case record and add it to the update set.

- **Trigger M2M:** You can get trigger relationships from your ServiceNow Studio scoped application by navigating to your PepsiCo Deliveries application scope. In the left navigation panel, look for "Trigger M2M" under AI Agent Studio components. Find the trigger relationships for your use case and add them to the update set.

- **AI Agents:** You can get AI agent definitions from your ServiceNow Studio scoped application by navigating to your PepsiCo Deliveries application scope. In the left navigation panel, look for "AI Agent" under AI Agent Studio components. Find both your Route Financial Analysis Agent and Route Decision Agent records and add them to the update set.

- **Agent Tools:** You can get agent tools from your ServiceNow Studio scoped application by navigating to your PepsiCo Deliveries application scope. In the left navigation panel, look for "Agent Tool" under AI Agent Studio components. Find all tools associated with your agents and add them to the update set.

- **Execution Tasks (Critical Step):** Navigate to System Definition > Tables, search for table name: `sn_aia_execution_plan`. Click on the table to open the list view. Look in the Objective column for plans related to your supply chain processing. Click on your execution plan record to open it. In the execution plan form, find the State field and click on the 'Completed' value (this is a clickable link). This opens a list view showing all tasks associated with your execution plan. Use Ctrl+A (Windows) or Cmd+A (Mac) to select all tasks, then right-click on the selected tasks > Add to Update Set.

### Evidence Records:
- Complete "dispatched" record from Delivery Delay table showing full workflow progression
- Sample record from Supply Agreement table demonstrating customer contract data
- Incident record created by Agent 1 during the financial analysis process

### n8n Integration:
- Exported n8n workflow configuration (`n8n-workflow.json`)
- n8n execution log: `n8n-execution.log` (consolidated input/output data from all workflow nodes)

**Required Log File:** `n8n-execution.log` - including:
- **Webhook node:** Capture INPUT data (received from ServiceNow)
- **AI Agent node:** Capture OUTPUT data (coordination decisions)
- **Logistics MCP Client:** Capture OUTPUT data (communications sent)
- **Retail MCP Client:** Capture OUTPUT data (communications sent)
- **ServiceNow MCP Client:** Capture OUTPUT data (status updates sent)

## 2. README.md Content Requirements

Required sections:

- **System Overview:** Description of the automated supply chain incident processing system and its business impact for PepsiCo operations

- **Implementation Steps:** Key architectural decisions, AI agent configuration choices, and integration approaches used

- **Architecture Diagram:** Visual representation of the complete workflow showing ServiceNow agents, n8n coordination, and external system integration

- **Optimization:** Analysis of how you optimized the system for efficiency, reliability, and performance. Document specific optimizations implemented (such as webhook URL configuration, script efficiency improvements, error handling enhancements, or workflow streamlining) and identify future optimization opportunities (such as caching strategies, parallel processing possibilities, advanced error recovery mechanisms, or enhanced monitoring capabilities).

- **Testing Results:** Evidence of successful end-to-end system operation with specific examples of financial analysis, routing decisions, and external execution

- **Business Value:** Analysis of how the system improves PepsiCo's supply chain operations, reduces manual intervention, and optimizes delivery cost management

## 3. Architecture Diagram Requirements

Create a comprehensive system flow diagram showing:
- Truck breakdown detection and ServiceNow agent processing
- AI-driven financial analysis and route decision workflows
- n8n orchestration of external system coordination
- Complete data flow and integration patterns
- Status progression and tracking capabilities

Use diagramming tool and save as `Diagram.png`
