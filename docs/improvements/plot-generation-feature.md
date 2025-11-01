# Plot Generation and Visualization Feature

**Date**: 2025-10-12
**Status**: Implemented
**Author**: Development Team

## Overview

This document describes the implementation of automated plot generation and data visualization capabilities in the Deep Research system. The feature enables the system to automatically generate relevant charts, graphs, and visualizations based on research findings.

## Motivation

Research reports often benefit from visual representations of data, especially when:
- Comparing multiple entities (companies, products, technologies)
- Showing trends over time
- Displaying distributions or market shares
- Illustrating correlations and relationships
- Presenting quantitative findings

Previously, the system only generated text-based reports. This enhancement allows the LLM to generate executable Python code for creating professional visualizations.

## Architecture Changes

### 1. Enhanced Final Report Generation Prompt

**File**: [`src/open_deep_research/prompts.py:255-272`](../../src/open_deep_research/prompts.py#L255-L272)

Added comprehensive instructions to the `final_report_generation_prompt_jz_prompt` that guide the LLM to:
- Generate Python visualization code when appropriate
- Use matplotlib, seaborn, or plotly libraries
- Include proper titles, labels, legends, and annotations
- Use appropriate chart types for different data (bar charts, line plots, scatter plots, etc.)
- Save visualizations as image files
- Provide explanations for each visualization

**Key Addition**:
```python
6. **IMPORTANT - Data Visualization**: When appropriate for the research topic,
   generate Python code to create relevant plots, charts, and visualizations that
   help illustrate key findings, trends, comparisons, or insights.
```

### 2. Visualization Planning Tool

**File**: [`src/open_deep_research/utils.py:250-292`](../../src/open_deep_research/utils.py#L250-L292)

Created a new tool `create_visualization()` that researcher agents can use during the research phase:

```python
@tool(description="Create data visualizations from research findings")
def create_visualization(
    visualization_type: str,
    data_description: str,
    chart_title: str,
    x_label: str = "",
    y_label: str = "",
) -> str:
    """Create data visualizations to illustrate research findings."""
```

**Purpose**: Allows researchers to plan visualizations as they discover quantitative data, creating a structured record that flows through to the final report generation.

### 3. Extended State Management

**File**: [`src/open_deep_research/state.py`](../../src/open_deep_research/state.py)

Added `visualization_plans` field to all relevant state classes:
- `AgentState` (line 72)
- `SupervisorState` (line 81)
- `ResearcherState` (line 93)
- `ResearcherOutputState` (line 100)

**State Definition**:
```python
visualization_plans: Annotated[list[str], override_reducer] = []
```

This enables visualization plans to be tracked and aggregated throughout the multi-agent workflow.

### 4. Updated Researcher Capabilities

**File**: [`src/open_deep_research/prompts.py:145-159`](../../src/open_deep_research/prompts.py#L145-L159)

Updated the `research_system_prompt_jz_prompt` to inform researchers about the visualization tool:

```python
<Available Tools>
You have access to three main tools:
1. **tavily_search**: For conducting web searches to gather information
2. **think_tool**: For reflection and strategic planning during research
3. **create_visualization**: For planning data visualizations based on quantitative research findings
```

Added guidance on when to use the tool (comparisons, trends, distributions, correlations).

### 5. Workflow Integration

**File**: [`src/open_deep_research/deep_researcher.py`](../../src/open_deep_research/deep_researcher.py)

#### Added Helper Function
```python
def get_visualization_plans_from_tool_calls(messages: list[MessageLikeRepresentation]):
    """Extract visualization plans from tool call messages."""
```

#### Modified compress_research()
- Lines 560-567: Extract visualization plans during research compression
- Lines 587-593: Include visualization plans in error cases

#### Updated supervisor_tools()
- Lines 333-339: Aggregate visualization plans from parallel researchers

#### Enhanced final_report_generation()
- Lines 637-651: Include visualization plans in the findings context passed to the report generation model

### 6. Tool Registration

**File**: [`src/open_deep_research/utils.py:627`](../../src/open_deep_research/utils.py#L627)

Added `create_visualization` to the toolkit available to researcher agents:

```python
tools = [tool(ResearchComplete), think_tool, create_visualization]
```

## Workflow Diagram

```
User Query
    ↓
Clarification & Research Brief
    ↓
Supervisor Agent (Plans Research Strategy)
    ↓
Spawns Multiple Parallel Researchers
    ↓
┌─────────────────────────────────────┐
│ Individual Researcher Agents:       │
│ - Search for information            │
│ - Gather quantitative data          │
│ - Call create_visualization() tool  │ ← NEW
│ - Document visualization plans      │ ← NEW
└─────────────────────────────────────┘
    ↓
Compress Research + Extract Viz Plans  ← NEW
    ↓
Supervisor Aggregates All Results
    ↓
Final Report Generation:
 - Written analysis
 - Python visualization code          ← NEW
 - Explanations of visualizations     ← NEW
    ↓
Output to User
```

## Example Usage

### User Query
```
"Compare the market share and growth trends of the top 3 cloud providers"
```

### Research Phase
Researchers gather data about AWS, Azure, and Google Cloud:
- Market share percentages
- Revenue growth rates
- Customer counts
- Regional distribution

A researcher calls:
```python
create_visualization(
    visualization_type="bar",
    data_description="Market share: AWS 32%, Azure 23%, Google Cloud 10%",
    chart_title="Cloud Provider Market Share 2024",
    y_label="Market Share (%)"
)
```

### Final Report Output
The report includes executable Python code:

```python
import matplotlib.pyplot as plt
import numpy as np

# Market share data from research
providers = ['AWS', 'Azure', 'Google Cloud']
market_share = [32, 23, 10]

# Create visualization
fig, ax = plt.subplots(figsize=(10, 6))
colors = ['#FF9900', '#0078D4', '#4285F4']
bars = ax.bar(providers, market_share, color=colors, alpha=0.8, edgecolor='black')

# Styling
ax.set_ylabel('Market Share (%)', fontsize=12, fontweight='bold')
ax.set_title('Cloud Provider Market Share 2024', fontsize=14, fontweight='bold', pad=20)
ax.set_ylim(0, 40)
ax.grid(axis='y', alpha=0.3, linestyle='--')

# Add value labels on bars
for bar in bars:
    height = bar.get_height()
    ax.text(bar.get_x() + bar.get_width()/2., height,
            f'{height}%', ha='center', va='bottom', fontweight='bold')

plt.tight_layout()
plt.savefig('cloud_market_share.png', dpi=300, bbox_inches='tight')
plt.show()
```

**Explanation**: This bar chart illustrates the dominant position of AWS in the cloud infrastructure market, with Azure as the second-largest provider and Google Cloud in third place.

## Implementation Details

### Data Flow

1. **Research Phase**:
   - Researcher agents discover quantitative data
   - Optional: Call `create_visualization()` to plan visualizations
   - Plans are stored in `ResearcherState.visualization_plans`

2. **Compression Phase**:
   - Research findings are compressed
   - Visualization plans are extracted using `get_visualization_plans_from_tool_calls()`
   - Both compressed research and visualization plans are returned in `ResearcherOutputState`

3. **Aggregation Phase**:
   - Supervisor receives results from all parallel researchers
   - Visualization plans are aggregated across all researchers
   - Plans are added to `SupervisorState.visualization_plans`

4. **Final Report Phase**:
   - All visualization plans are included in the findings context
   - The enhanced prompt instructs the LLM to generate appropriate Python code
   - Code is embedded in markdown code blocks in the final report

### Backward Compatibility

The implementation is fully backward compatible:
- If no visualizations are planned, the system works exactly as before
- The `visualization_plans` fields default to empty lists
- The prompt additions only take effect when relevant data exists

### Supported Visualization Types

The system can generate code for various chart types:
- **Bar charts**: Comparisons between categories
- **Line plots**: Trends over time
- **Scatter plots**: Correlations and relationships
- **Pie charts**: Proportions and distributions
- **Heatmaps**: Multi-dimensional data
- **Box plots**: Statistical distributions
- **Radar charts**: Multi-attribute comparisons
- **Area charts**: Cumulative trends
- **Histograms**: Frequency distributions

### Library Support

Generated code uses standard Python visualization libraries:
- **matplotlib**: Core plotting functionality
- **seaborn**: Statistical visualizations
- **plotly**: Interactive plots (when appropriate)
- **pandas**: Data manipulation (when needed)

## Benefits

1. **Enhanced Understanding**: Visual representations make complex data more accessible
2. **Professional Reports**: Publication-ready charts and graphs
3. **Flexible Output**: Code can be executed in various environments (Jupyter, scripts, notebooks)
4. **Customizable**: Users can modify the generated code for their specific needs
5. **Reproducible**: Self-contained code with all data definitions included
6. **Multi-Agent Coordination**: Visualizations can be planned across multiple research streams

## Future Enhancements

Potential improvements for future versions:

1. **Automatic Code Execution**: Run the visualization code and embed images directly
2. **Interactive Dashboards**: Generate plotly/Dash dashboards for interactive exploration
3. **Style Templates**: Configurable visualization styles (corporate, academic, etc.)
4. **Data Export**: Export underlying data as CSV/JSON for further analysis
5. **3D Visualizations**: Support for 3D plots and advanced visualizations
6. **Geographic Maps**: Integration with mapping libraries for location-based data
7. **Animation Support**: Time-series animations for trend visualization

## Testing Recommendations

To test this feature:

1. **Comparison Query**: "Compare the revenue and user base of Netflix, Disney+, and Hulu"
   - Should generate bar charts comparing metrics

2. **Trend Query**: "Show the growth trend of AI research papers from 2020-2024"
   - Should generate line plots showing trends

3. **Market Analysis**: "Analyze the smartphone market share distribution"
   - Should generate pie charts or stacked bar charts

4. **Correlation Study**: "Research the relationship between R&D spending and innovation output"
   - Should generate scatter plots with trend lines

## Configuration

No additional configuration is required. The feature is enabled by default. However, ensure:

1. The research model supports code generation in responses
2. Sufficient token limits for including visualization code in reports
3. The `final_report_model` is capable of generating well-formatted Python code

## Dependencies

All required libraries are already included in the project dependencies:
- `matplotlib` (via transitive dependencies)
- Standard Python data manipulation libraries

Users executing the generated code will need:
```bash
pip install matplotlib seaborn plotly pandas numpy
```

## Files Modified

| File | Lines | Changes |
|------|-------|---------|
| `prompts.py` | 255-272 | Added visualization instructions to final report prompt |
| `prompts.py` | 145-159 | Updated researcher prompt to include visualization tool |
| `utils.py` | 250-292 | Created `create_visualization()` tool |
| `utils.py` | 627 | Added tool to researcher toolkit |
| `utils.py` | 651-666 | Added `get_visualization_plans_from_tool_calls()` helper |
| `state.py` | 72, 81, 93, 100 | Added `visualization_plans` to state classes |
| `deep_researcher.py` | 49 | Imported visualization helper function |
| `deep_researcher.py` | 560-567, 587-593 | Extract visualization plans in compress_research |
| `deep_researcher.py` | 333-339 | Aggregate visualization plans in supervisor_tools |
| `deep_researcher.py` | 637-651 | Include visualization plans in final report context |

## Conclusion

This enhancement significantly improves the Deep Research system's ability to communicate quantitative findings through visual representations. The implementation maintains the system's multi-agent architecture while adding a new dimension of data presentation that makes research reports more comprehensive and accessible.
