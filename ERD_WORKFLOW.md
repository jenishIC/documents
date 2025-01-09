# ERD Relationship Validator - Workflow Guide

## Overview

This tool helps validate relationships between marketing datasets by combining traditional ERD extraction with AI-powered validation. It's particularly useful for understanding connections between different marketing and sales platforms in your data stack.

## Quick Start

1. Install Dependencies:
   ```bash
   pip install -r requirements_marketing.txt
   ```

2. Set Environment Variables:
   ```bash
   export OPENAI_API_KEY="your-openai-key"
   export GOOGLE_APPLICATION_CREDENTIALS="path/to/credentials.json"
   ```

3. Run Basic Analysis:
   ```python
   from erd_extract import SchemaRelationshipAnalyzer
   
   analyzer = SchemaRelationshipAnalyzer("your-project-id")
   results = analyzer.analyze_datasets(["ROM", "zGA4", "zHubspot"])
   ```

## Workflow Steps

### 1. Data Source Preparation

The analyzer works with these marketing datasets:
- ROM (Raw Object Model)
- Sales funnel data (sales_funnel_dashboard, sales_ops_funnel)
- Marketing platforms (zGA4, zGoogleAds, zHubspot)
- Sales platforms (zSalesforce, zSalesloft)
- Customer support (zZendesk)
- Additional sources (z6sense, zNooks, etc.)

### 2. Relationship Detection Process

```
Step 1: Schema Analysis
- Extracts column names and types
- Builds initial relationship candidates

Step 2: Base Similarity Check
- Computes embedding-based similarity
- Checks type compatibility
- Identifies potential matches

Step 3: LLM Validation (for marketing datasets)
- Validates relationships using AI
- Considers marketing context
- Provides detailed reasoning

Step 4: Results Generation
- Combines similarity scores
- Sorts by confidence
- Groups by source tables
```

### 3. Understanding Results

Example output structure:
```python
{
    'relationships': [
        {
            'source_table': 'zGA4.events',
            'source_column': 'visitor_id',
            'target_table': 'zHubspot.contacts',
            'target_column': 'anonymous_id',
            'confidence': 0.92,
            'reasoning': 'Valid relationship: connects anonymous visitor tracking to lead identification'
        },
        # More relationships...
    ]
}
```

### 4. Common Usage Patterns

#### A. Basic Marketing Flow Analysis
```python
# Analyze core marketing-to-sales flow
analyzer.analyze_datasets([
    "zGA4",          # Traffic source
    "zHubspot",      # Lead management
    "zSalesforce"    # Sales process
])
```

#### B. Full Customer Journey Analysis
```python
# Analyze complete customer journey
analyzer.analyze_datasets([
    "zGA4",          # Initial visit
    "zGoogleAds",    # Ad interaction
    "zHubspot",      # Lead capture
    "zSalesloft",    # Sales engagement
    "zSalesforce",   # Opportunity management
    "zZendesk"       # Customer support
])
```

#### C. Custom Analysis
```python
# Analyze specific datasets with stricter threshold
analyzer = SchemaRelationshipAnalyzer(
    project_id="your-project-id",
    threshold=0.90
)
results = analyzer.analyze_datasets([
    "ROM",
    "sales_funnel_dashboard"
])
```

## Best Practices

1. **Dataset Selection**
   - Start with core marketing datasets
   - Add related datasets incrementally
   - Group analysis by business function

2. **Relationship Validation**
   - Review high-confidence matches first
   - Check reasoning for marketing context
   - Verify temporal sequence logic

3. **Performance Optimization**
   - Use parallel processing for large datasets
   - Cache results for repeated analyses
   - Monitor memory usage with large schemas

## Common Use Cases

1. **Marketing Attribution**
   - Track user journey across platforms
   - Connect ad interactions to leads
   - Map conversion paths

2. **Sales Pipeline Analysis**
   - Link marketing leads to opportunities
   - Track deal progression
   - Connect customer interactions

3. **Customer Journey Mapping**
   - Map complete user journey
   - Identify cross-platform touchpoints
   - Analyze support interactions

## Troubleshooting

1. **Low Confidence Scores**
   - Check column name similarity
   - Verify type compatibility
   - Review marketing context

2. **Missing Relationships**
   - Verify dataset accessibility
   - Check schema extraction
   - Adjust similarity threshold

3. **Performance Issues**
   - Reduce dataset scope
   - Clear embedding cache
   - Monitor memory usage

## Next Steps

1. Review the technical documentation in README_ERD_TECHNICAL.md for detailed implementation details
2. Customize the marketing context based on your specific business flow
3. Adjust thresholds and parameters based on your data patterns
