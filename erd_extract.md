# ERD Extractor with LLM Validation - Technical Documentation

## Architecture Overview

### 1. Core Components

#### 1.1 SchemaRelationshipAnalyzer
- Primary class that orchestrates the relationship detection process
- Manages both embedding-based and LLM-based validation
- Handles parallel processing of datasets and relationships

#### 1.2 ValidateMarketingRelationship
- DSPY Signature class for LLM validation
- Defines input/output contract for LLM interactions
- Processes marketing-specific relationship validation

### 2. Data Flow

```
[BigQuery Tables] → [Schema Extraction] → [Relationship Detection] → [Validation] → [Results]
     ↓                      ↓                       ↓                    ↓
   Tables              Column Types          Potential Pairs        Final Scores
   Schemas             Field Names           Similarity Scores      Reasoning
```

## Technical Implementation Details

### 1. Initialization Process

```python
def __init__(self, project_id: str, threshold: float = 0.85):
    # Core components initialization
    self.client = bigquery.Client(project=project_id)
    self.model = SentenceTransformer('all-MiniLM-L6-v2')
    self.embedding_cache = {}
    
    # LLM setup
    if os.getenv("OPENAI_API_KEY"):
        dspy.settings.configure(lm=dspy.OpenAI())
        self.validator = dspy.Predict(ValidateMarketingRelationship)
        self.use_llm = True
```

### 2. Schema Processing

#### 2.1 Parallel Schema Extraction
```python
with concurrent.futures.ThreadPoolExecutor() as executor:
    future_to_table = {
        executor.submit(self.get_table_schema, dataset_id, table.table_id): table.table_id
        for table in tables
    }
```

#### 2.2 Schema Structure
```python
{
    'dataset.table_name': {
        'fields': {
            'column_name': 'data_type',
            # ... more columns
        },
        'row_count': int
    }
}
```

### 3. Relationship Detection Pipeline

#### 3.1 Base Similarity Calculation
1. Embedding Generation:
   ```python
   def get_embedding(self, text: str) -> np.ndarray:
       if text not in self.embedding_cache:
           self.embedding_cache[text] = self.model.encode([text])[0]
       return self.embedding_cache[text]
   ```

2. Similarity Computation:
   ```python
   base_similarity = cosine(emb1, emb2)
   if col1.lower() == col2.lower():
       base_similarity = max(base_similarity, 0.95)
   ```

#### 3.2 Type Compatibility Check
```python
compatible_types = {
    'STRING': {'STRING'},
    'INTEGER': {'INTEGER', 'FLOAT', 'NUMERIC'},
    'FLOAT': {'FLOAT', 'INTEGER', 'NUMERIC'},
    'NUMERIC': {'NUMERIC', 'INTEGER', 'FLOAT'},
    'TIMESTAMP': {'TIMESTAMP', 'DATETIME'},
    'DATETIME': {'DATETIME', 'TIMESTAMP'},
}
```

#### 3.3 LLM Validation Process
1. Context Creation:
   ```python
   relationship_context = f"""
   Analyzing relationship between:
   Table 1: {table1} (Column: {col1})
   Table 2: {table2} (Column: {col2})
   Base similarity score: {base_similarity}
   
   {self.marketing_context}
   """
   ```

2. LLM Prediction:
   ```python
   prediction = self.validator(
       context=relationship_context,
       table1=table1,
       column1=col1,
       table2=table2,
       column2=col2
   )
   ```

3. Score Combination:
   ```python
   final_similarity = (base_similarity + prediction.confidence) / 2
   ```

### 4. Parallel Processing Strategy

#### 4.1 Dataset Level Parallelization
```python
with concurrent.futures.ThreadPoolExecutor() as executor:
    future_to_dataset = {
        executor.submit(self.process_dataset, dataset_id): dataset_id
        for dataset_id in datasets
    }
```

#### 4.2 Table Pair Processing
```python
with concurrent.futures.ProcessPoolExecutor() as executor:
    all_relationships = list(executor.map(process_table_pair, table_pairs))
```

### 5. Result Structure

```python
{
    'table_schemas': {
        # Full schema information
    },
    'relationships': [
        {
            'source_table': str,
            'source_column': str,
            'target_table': str,
            'target_column': str,
            'confidence': float,
            'reasoning': str
        }
    ],
    'grouped_relationships': {
        'table_name': [
            # Relationships where table_name is source
        ]
    }
}
```

## Performance Considerations

### 1. Caching Strategy
- Embeddings are cached to avoid recomputation
- Schema information is stored in memory during analysis
- LLM calls are made only for marketing datasets

### 2. Parallelization Points
- Dataset processing: ThreadPoolExecutor
- Table pair analysis: ProcessPoolExecutor
- Schema extraction: Async operations

### 3. Memory Management
- Embedding cache grows with unique column names
- Relationship results are stored in memory
- Grouped relationships maintain references

## Error Handling

### 1. LLM Fallback
```python
try:
    # LLM validation
    prediction = self.validator(...)
except Exception as e:
    print(f"LLM validation failed: {e}. Using base similarity.")
    return base_similarity, "Using base similarity due to LLM error"
```

### 2. Schema Processing Errors
```python
try:
    schema = future.result()
    table_schemas[f"{dataset_id}.{table_id}"] = schema
except Exception as exc:
    print(f"Error processing table {table_id}: {exc}")
```

## Usage Examples

### 1. Basic Usage
```python
analyzer = SchemaRelationshipAnalyzer("your-project-id")
results = analyzer.analyze_datasets([
    "ROM",
    "zGA4",
    "zHubspot"
])
```

### 2. Custom Threshold
```python
analyzer = SchemaRelationshipAnalyzer(
    project_id="your-project-id",
    threshold=0.90  # More strict matching
)
```

### 3. Accessing Results
```python
# Get all relationships
relationships = results['relationships']

# Get relationships for specific table
table_rels = results['grouped_relationships']['table_name']

# Get schema information
schemas = results['table_schemas']
```

## Dependencies

- google-cloud-bigquery: BigQuery client
- sentence-transformers: Embedding generation
- numpy: Numerical operations
- scipy: Similarity calculations
- dspy-ai: LLM integration
- pandas: Data manipulation
- concurrent.futures: Parallel processing
