# PepsiCo Supply Chain Incident Processing - ServiceNow Implementation

## Overview

**Company Context:** You're a ServiceNow AI Systems Developer at PepsiCo, where reliable supply chain operations are critical for delivering products to major retail partners like Whole Foods. When delivery trucks experience breakdowns, rapid financial analysis and optimal routing decisions are essential to minimize customer impact and contractual penalties.

**Your Role:** As a ServiceNow AI Systems Developer at PepsiCo, you've been tasked with building an automated supply chain incident processing system that analyzes financial impacts of truck breakdowns, makes optimal routing decisions, and coordinates external execution through AI agents and workflow orchestration.

**The Business Problem:** PepsiCo's logistics operations need an intelligent system that automatically calculates delay costs based on customer contracts, selects optimal rerouting options considering both financial impact and delivery constraints, and coordinates execution with external logistics providers and customer notifications without manual intervention.

## Prerequisites

**Required Knowledge:** Understanding of ServiceNow AI Agent Studio, basic n8n workflow design, and system integration concepts.

## System Resources and Context

### Requirements:
- Bearer token for ServiceNow MCP authentication from n8n environment
- AWS Bedrock access keys for AI model integration in n8n
- JavaScript code for ServiceNow agent to kick off n8n workflow through webhook

### What I Built:
- ServiceNow scoped application with AI agents for supply chain processing
- n8n workflow for external system coordination
- Complete integration testing and validation

**n8n Environment Setup:** I created a free n8n account with my personal a new email address 
## Assignment Objectives

### AI Agent System Component Flow
<img width="1657" height="1403" alt="Diagram png" src="https://github.com/user-attachments/assets/0dfdd9fa-d465-4a12-a9b1-c852fca8711a" />


### Full System Components:
<img width="631" height="286" alt="image" src="https://github.com/user-attachments/assets/25183fae-db47-432d-b2fa-efb56fdebe97" />


1. Logistics provider (Schneider) detects truck breakdown and updates PepsiCo ServiceNow via MCP protocol
2. ServiceNow use case with two sequential AI agents processes breakdown and calculates optimal routing
3. n8n receives routing decisions via webhook and coordinates with external logistics and customer systems
4. Status updates flow back through MCP to provide execution tracking

## Implementation Objectives

### Build Intelligent Financial Analysis
- I created an AI agent that calculates delay costs based on delivery contracts
- I implement mathematical analysis of route options with penalty calculations
- I generate comprehensive financial impact assessments in a structured format

### Enable Smart Route Decision Making
- I developed an AI agent that selects optimal routes based on cost and time constraints
- I configured agent instructions for route selection and incident priority assignment through natural language prompts
- I created a trigger for external execution workflows through webhook integration

### Coordinate External System Integration
- I built an n8n agent workflow that receives routing decisions via webhook
- I integrate with multiple external systems using MCP protocol
- I ensure proper status updates and execution tracking

## Sequential Configuration Steps

### Step 1: Application Setup
I created a scoped application with the exact name: **PepsiCo Deliveries**

*This precise naming auto-generate the scope: x_snc_pepsico_de_0*
<img width="794" height="371" alt="image" src="https://github.com/user-attachments/assets/391d79fb-5f6b-4ab2-bf3d-c312a00a6d85" />

### Step 2: Tables' Structure Requirements

#### Delivery Delay Table: I create a table with the name "Delivery Delay"

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

**Important:** The `assigned_to` field in the Delivery Delay table serves as the execution context for AI agent triggers. This ensures proper permissions and security boundaries when agents process records automatically.

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

#### Supply Agreement Table: I create a table with the name "Supply Agreement"

**Required fields:**
- `customer_id` (Integer, Primary Key)
- `customer_name` (String, 100)
- `deliver_window_hours` (Integer) - Contractual delivery timeframe
- `stockout_penalty_rate` (Integer) - Cost per hour of delay in dollars
<img width="793" height="303" alt="image" src="https://github.com/user-attachments/assets/aff9698a-bc25-4cd7-b55a-54f878621272" />

**Required Sample Record:** I created this record in the Supply Agreement table:
- `customer_id`: 1
- `customer_name`: "Whole Foods"
- `deliver_window_hours`: 3
- `stockout_penalty_rate`: 250
<img width="799" height="80" alt="image" src="https://github.com/user-attachments/assets/206c2643-9ead-4c59-8080-7b022658c190" />

### Data Dictionary

- **`deliver_window_hours`**: The contractual timeframe (in hours) within which deliveries must be completed to avoid penalties. For Whole Foods, deliveries must be completed within 3 hours of departure to avoid charges.

- **`stockout_penalty_rate`**: The financial penalty (in dollars) assessed for every hour a delivery exceeds the contractual delivery window. Whole Foods charges PepsiCo $250 for each hour beyond the 3-hour delivery window. For example, a 5-hour delivery would incur penalties for 2 hours (5 minus 3), resulting in a $500 penalty charge.

### Step 3: Use Case and Trigger Configuration
<img width="856" height="351" alt="image" src="https://github.com/user-attachments/assets/0f7e7cc0-ec9e-467e-abdc-7535c0101c8f" />

**Critical Requirement:** I created a use case with a trigger configuration that activates when:
- **Table:** Delivery Delay
- **Condition:** Status equals "pending"
- **Run As:** User referenced in the assigned_to field of the triggering record
<img width="828" height="152" alt="image" src="https://github.com/user-attachments/assets/14385dbd-2ace-4891-a7c9-15ad81c981b6" />

**Trigger Testing:** If automatic triggering doesn't function correctly, test your agents manually through AI Agent Studio using specific route_id values to validate functionality. Document why your trigger may not be working.

### Step 4: AI Agent Architecture

#### Agent 1: Route Financial Analysis Agent

**Purpose:** Analyze the financial impact of delivery disruptions and create incident tracking
<img width="947" height="269" alt="image" src="https://github.com/user-attachments/assets/16d57e9b-c71f-428b-98c9-b91609c318da" />


<img width="935" height="240" alt="image" src="https://github.com/user-attachments/assets/77a39b2e-457d-41a6-a898-d4201b95e9e4" />


<img width="938" height="256" alt="image" src="https://github.com/user-attachments/assets/b5b8ea10-f9f6-4afd-b8a9-6a4255a8f70b" />

**Suggested Tools:**
- Record lookup operations for delivery delay and supply agreement data
- Incident creation and management tools
- Status update operations for workflow progression
<img width="949" height="290" alt="image" src="https://github.com/user-attachments/assets/eb4b6bd1-9373-426c-8aed-b583c9c3fc83" />

**Key Responsibilities:**
- Query customer contract terms and penalty structures from the Supply Agreement table
- Calculate delay costs for each proposed route option using `deliver_window_hours` and `stockout_penalty_rate`
- Create incident records with comprehensive breakdown details
- Update delivery records with calculated financial impact
- Progress workflow status to trigger next agent

#### Agent 2: Route Decision Agent

**Purpose:** Select optimal routes and coordinate external execution
<img width="946" height="208" alt="image" src="https://github.com/user-attachments/assets/aa569b62-b2ea-4b7e-9f24-5dab3e186860" />
<img width="935" height="226" alt="image" src="https://github.com/user-attachments/assets/488da86d-fff4-4121-815a-42622f00c7f8" />
<img width="788" height="173" alt="image" src="https://github.com/user-attachments/assets/9f776694-c5e8-4902-87cf-795deadc2a1a" />
<img width="757" height="143" alt="image" src="https://github.com/user-attachments/assets/a54a69f0-7dee-41c5-8ac8-7e2b3183378d" />

**Suggested Tools:**
- Record lookup operations for calculated financial data
- Route selection and update operations
- Incident priority management tools
- Webhook or script tools for n8n integration

<img width="949" height="331" alt="image" src="https://github.com/user-attachments/assets/d61b280d-8a9f-42e8-bc3e-990e083f6655" />

**Key Responsibilities:**
- Analyze route options using financial impact and delivery constraints
- Select optimal routing based on business rules and cost optimization
- Update incident priority based on financial severity
- Trigger external execution workflow via webhook to n8n
- Update workflow status to approved for tracking

**Tool Implementation Flexibility:** Choose between ServiceNow record operations or custom script implementations for CRUD operations. Both approaches are acceptable as long as they achieve the required functionality.

**Important Script Configuration:** When implementing script tools for webhook integration, configure the webhook URL as a variable at the top of your script for easy maintenance and proper logging. This ensures consistent URL management and improves script reliability.

### Step 5: n8n Workflow Implementation and Logging
<img width="482" height="247" alt="image" src="https://github.com/user-attachments/assets/b910f520-d398-4657-9b61-c080dc5feb66" />

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

I captured execution logs for all workflow components and consolidated them into a single log file. (Located above)

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
<img width="780" height="298" alt="image" src="https://github.com/user-attachments/assets/7577c792-865f-4663-821c-de4c4333492a" />

## Critical Success Factors

- AI agents must successfully process delivery delays and calculate accurate financial impacts
- Route selection must demonstrate intelligent optimization based on cost and delivery constraints
- n8n workflow must successfully coordinate external system integration via MCP protocol
- System must handle error conditions gracefully and provide meaningful feedback
- Documentation must clearly explain the business value and technical implementation approach

**Important Troubleshooting Note:** If the automatic trigger doesn't activate with new "pending" records, validate your use case configuration and test manually through AI Agent Studio to ensure core functionality works correctly.

## System Optimization Summary
 
I refined this system to make it run faster, stay consistent, and handle errors more smoothly. The focus was on maintaining a clean data flow between ServiceNow, n8n, and the AI agents, without unnecessary steps.

**Webhook Fix:** Updated the webhook to the live production URL, ensuring an instant and reliable connection.

**Simplified Flow:** Removed extra logic so the process moves directly from the webhook to the AI agent and back into ServiceNow.

**Smarter Script:** Replaced multiple API calls with one clean n8n webhook script, which cut down on time and complexity.

**Error Recovery:** Added fallback checks and clear success messages so the system keeps running even if something fails.

**Better Tracking:** Every run is logged for quick troubleshooting and easy visibility into what happened.

**Faster Performance:** Shared data between agents instead of reloading it, which sped up response times and reduced duplicate work.

**Next Steps:**
Add caching to avoid repeated lookups, run agents side-by-side for faster results, and include automated retries and better monitoring to make it even more reliable.

## Testing Results: 
Evidence of successful end-to-end system operation with specific examples of financial analysis, routing decisions, and external execution
<img width="956" height="370" alt="image" src="https://github.com/user-attachments/assets/0f3d768d-602e-4d38-b486-85c2931d2f6a" />
<img width="948" height="341" alt="image" src="https://github.com/user-attachments/assets/89a545d2-4ab7-404f-b1a5-c59542a45a6e" />
<img width="287" height="248" alt="image" src="https://github.com/user-attachments/assets/7eadb825-da61-4844-b298-03f920b66de9" />
<img width="274" height="291" alt="image" src="https://github.com/user-attachments/assets/3ad2f844-1f18-4ed6-a578-a224084cd3be" />
<img width="256" height="328" alt="image" src="https://github.com/user-attachments/assets/ecc60e52-b3cb-42a7-806b-a11439e52dc0" />
<img width="246" height="340" alt="image" src="https://github.com/user-attachments/assets/ac746890-def0-40ce-9c7a-db6e3c8b35d8" />
<img width="265" height="317" alt="image" src="https://github.com/user-attachments/assets/31359377-0184-4e68-a6b4-cd298f9e0287" />
<img width="265" height="287" alt="image" src="https://github.com/user-attachments/assets/202af251-160e-4146-b5c8-932ea8461bf5" />

<img width="257" height="113" alt="image" src="https://github.com/user-attachments/assets/660517fc-b3d2-4f60-a42e-7c3e5c115fae" />





## Business Value:
 This system brings measurable efficiency, accuracy, and insight to PepsiCo's supply chain operations that once depended on manual updates and spreadsheets. By linking ServiceNow, n8n, and AI-powered agents, it automates the full cycle of delivery tracking from detecting late shipments to calculating penalties and creating incidents in real time.

Before automation, a delayed delivery required analysts to pull data, check contract terms, calculate fines, and manually open an incident ticket. Now those same actions happen instantly. The workflow fetches the route record, reads the customer’s Supply Agreement, calculates the delay cost, and pushes an Incident with all the details attached to the right record in seconds.

This saves hours of coordination for each delay, eliminates data entry mistakes, and ensures financial accuracy when penalties affect profit margins. Managers gain faster visibility into performance trends, seeing which routes or vendors are driving cost overruns, and can act immediately instead of waiting for weekly reports.

The automation also improves customer trust because deliveries are monitored continuously, response times are faster, and incident updates are consistent across teams. Finance teams get real-time cost reporting, logistics gains proactive alerts, and operations can focus on prevention instead of cleanup.

Long term, the same architecture can extend beyond logistics by automating SLA monitoring for procurement, IT service management, or vendor compliance. It is not just about cutting steps; it is about turning reactive processes into predictive, data-driven workflows that save time, reduce costs, and scale with the business.

