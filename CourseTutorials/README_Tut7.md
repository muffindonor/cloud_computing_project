
# Search Services System

A modular system for document search and management designed to run on Google Colab.

## Architecture

The system consists of four main services:

```
┌────────────────┐     ┌─────────────────┐     ┌────────────────┐
│  IndexService  │────>│  QueryService   │────>│  ResultService │
└────────────────┘     └─────────────────┘     └────────────────┘
        │                      │                       │
        │                      ▼                       │
        │              ┌─────────────────┐            │
        └────────────>│  RankingService  │<───────────┘
                      └─────────────────┘
```

### System Services

1. **IndexService**
   - Document management and search indexing
   - Creation of inverted word index
   - Document addition and retrieval

2. **QueryService**
   - Search query processing
   - Support for logical operators
   - Integration with RankingService for scoring

3. **RankingService**
   - Relevance score calculation
   - Search result ranking
   - Advanced weighting algorithm

4. **ResultService**
   - Result formatting
   - Ranking information addition
   - Snippet generation

## Ranking Algorithm

RankingService uses an algorithm that weighs multiple parameters:

1. **Search Term Frequency**:
   ```python
   # Calculate content occurrences
   content_count = content.count(term.lower())
   
   # Calculate title occurrences (double weight)
   title_count = title.count(term.lower()) * 2
   
   # Final term score
   term_score = title_count + content_count
   ```

2. **Weights**:
   - Title occurrences: double weight (×2)
   - Content occurrences: normal weight (×1)

3. **Final Score Calculation**:
   ```python
   final_score = sum(term_scores)
   ```

Calculation Example:
- Document with "python" appearing twice in title and once in content:
  - Title score: 2 × 2 = 4
  - Content score: 1 × 1 = 1
  - Final score: 4 + 1 = 5

## Basic Usage

### Creating Documents

```python
# Create services
index_service = IndexService()
query_service = QueryService(index_service)
result_service = ResultService(index_service, query_service)

# Add a document
doc = index_service.add_document({
    'title': 'Python Programming',
    'content': 'Python is a popular programming language'
})
```

### Performing Searches

```python
# AND search
query = query_service.create_query({
    'expression': ['python', 'AND', 'programming']
})

# OR search
query = query_service.create_query({
    'expression': ['python', 'OR', 'cloud']
})

# Get formatted results
results = result_service.format_results(query['id'])
```

## Advanced Examples

### Complex Search

```python
# Combined search
query = query_service.create_query({
    'expression': ['python', 'AND', 'web', 'OR', 'cloud']
})

# Display ranked results
results = result_service.format_results(query['id'])
for result in results['formatted_results']:
    print(f"Title: {result['title']}")
    print(f"Score: {result['relevance_score']}")
    print(f"Snippet: {result['snippet']}\n")
```

### Sample Output

```
Title: Web Development with Python
Score: 4
Snippet: Python frameworks are popular for web development...

Title: Cloud Services Architecture
Score: 3
Snippet: Cloud computing enables scalable services...

Title: Python Programming
Score: 2
Snippet: Python is a popular programming language...
```

## Testing

```python
def test_search():
    # Initialize services
    index_service = IndexService()
    query_service = QueryService(index_service)
    result_service = ResultService(index_service, query_service)

    # Add test document
    doc = index_service.add_document({
        'title': 'Test Document',
        'content': 'This is a test document about microservices'
    })
    
    # Test search
    query = query_service.create_query({
        'expression': ['test', 'AND', 'microservices']
    })
    
    # Test results
    results = result_service.format_results(query['id'])
    assert results['count'] == 1
    assert results['formatted_results'][0]['relevance_score'] > 0
```
