# Custom Query Builder Documentation

## Overview
The Custom Query Builder is a React-based component that allows users to build complex SQL queries through a user-friendly interface. It supports filtering Salesforce data with both property and relation-based filters.

## Component Architecture

### 1. Component Hierarchy
```
AudienceQueryBuilder
├── CustomQueryBuilder (Properties)
│   ├── RuleGroup
│   │   └── RuleComponent
└── CustomQueryBuilder (Relations)
    ├── RuleGroup
    │   └── RuleComponent
```

### 2. Component Responsibilities

#### AudienceQueryBuilder
- Main container component
- Manages state for both property and relation filters
- Handles column data formatting
- Processes joins for relation filters
- Provides filtered columns to child components

#### CustomQueryBuilder
- Renders filter groups
- Manages group-level operations
- Handles rule additions and deletions
- Generates SQL queries

#### RuleGroup
- Renders group header with AND/OR conditions
- Manages rules within the group
- Handles group-level operations
- Provides visual hierarchy with vertical lines

#### RuleComponent
- Renders individual filter rules
- Handles field selection
- Manages operators and values
- Provides field-specific input types

## State Management

### 1. Group State Structure
```javascript
const [groups, setGroups] = useState([
  {
    id: 'relations-group',
    condition: 'and',
    rules: [],
    isRelationship: true,
    label: 'Relations Filter'
  },
  {
    id: 'properties-group',
    condition: 'and',
    rules: [],
    isPropertyGroup: true,
    label: 'Properties Filter'
  }
]);
```

### 2. State Update Flow
```javascript
// When adding a rule
handleAddRule = (groupId) => {
  setGroups(prevGroups => {
    const newGroups = prevGroups.map(group => {
      if (group.id === groupId) {
        return {
          ...group,
          rules: [...group.rules, newRule]
        };
      }
      return group;
    });
    return newGroups;
  });
};

// When deleting a rule
handleDeleteRule = (groupId, ruleIndex) => {
  setGroups(prevGroups => {
    const newGroups = prevGroups.map(group => {
      if (group.id === groupId) {
        return {
          ...group,
          rules: group.rules.filter((_, idx) => idx !== ruleIndex)
        };
      }
      return group;
    });
    return newGroups;
  });
};
```

### 3. Filter Deletion Workflow
1. User initiates rule deletion
2. Rule is removed from group's rules array
3. Group is checked for remaining rules
4. If group is relation group and has no rules:
   - All joins for that relation are removed
   - Query is regenerated without the joins
5. State is updated and UI reflects changes

## Data Flow

### 1. Initial Setup
```javascript
// Column Configuration
const formattedColumnsData = [
  {
    field: 'Account.Name',        // Field identifier
    label: 'Account Name',        // Display label
    type: 'string',              // Data type
    joinInfo: {                  // Relation information
      sourceTable: 'Account',
      sourceKey: 'id',
      targetTable: 'Contact',
      targetKey: 'account_id'
    }
  }
];

// Group Structure
const groups = [
  {
    id: 'properties-group',
    condition: 'and',
    rules: [],
    isPropertyGroup: true
  },
  {
    id: 'relations-group',
    condition: 'and',
    rules: [],
    isRelationship: true
  }
];
```

### 2. Rule Structure
```javascript
const rule = {
  field: 'Account.Name',     // Field identifier
  operator: 'contains',      // Operation type
  value: 'Tech',            // Filter value
  type: 'string'            // Data type
};
```

### 3. Generated Query Structure
```sql
-- With relation filters
SELECT DISTINCT account.*
FROM salesforce_account account
LEFT JOIN salesforce_contact contact ON contact.account_id = account.id
WHERE account.name LIKE '%Tech%'
  AND contact.email LIKE '%gmail%'

-- Without relation filters
SELECT DISTINCT account.*
FROM salesforce_account account
WHERE account.name LIKE '%Tech%'
```

## Configuration Guide

### 1. Model Configuration
```javascript
// Define your models with relationships
const models = {
  Account: {
    table: 'salesforce_account',
    primaryKey: 'id',
    fields: ['name', 'type', 'industry'],
    relationships: {
      contacts: {
        model: 'Contact',
        type: 'hasMany',
        foreignKey: 'account_id'
      },
      opportunities: {
        model: 'Opportunity',
        type: 'hasMany',
        foreignKey: 'account_id'
      }
    }
  },
  Contact: {
    table: 'salesforce_contact',
    primaryKey: 'id',
    fields: ['email', 'phone'],
    relationships: {
      account: {
        model: 'Account',
        type: 'belongsTo',
        foreignKey: 'account_id'
      }
    }
  }
};
```

### 2. Column Configuration
```javascript
// Convert models to column configuration
const generateColumns = (models) => {
  const columns = [];
  
  Object.entries(models).forEach(([modelName, model]) => {
    // Add fields
    model.fields.forEach(field => {
      columns.push({
        field: `${modelName}.${field}`,
        label: `${modelName} ${field}`,
        type: getFieldType(field),
        joinInfo: null
      });
    });

    // Add relationship fields
    Object.entries(model.relationships).forEach(([relationName, relation]) => {
      columns.push({
        field: `${relation.model}.${relationName}`,
        label: `${relation.model} ${relationName}`,
        type: 'relation',
        joinInfo: {
          sourceTable: model.table,
          sourceKey: model.primaryKey,
          targetTable: models[relation.model].table,
          targetKey: relation.foreignKey
        }
      });
    });
  });

  return columns;
};
```

### 3. Usage Example
```javascript
// Initialize the query builder
const AudienceFilterExample = () => {
  const columns = generateColumns(models);

  const handleRuleChange = (rules) => {
    // rules.groups - Contains all filter groups
    // rules.joins - Contains necessary joins
    // rules.cleanedGroups - Contains processed groups
    
    // Generate and execute query
    const query = generateWhereClause(rules);
    executeQuery(query);
  };

  return (
    <AudienceQueryBuilder
      formattedColumnsData={columns}
      handleRuleChange={handleRuleChange}
    />
  );
};
```

## Query Generation Process

1. **Rule Processing**
   - Validate rules (field, operator, value)
   - Convert rules to SQL conditions
   - Group conditions by their operators (AND/OR)

2. **Join Generation**
   - Identify required joins from relation filters
   - Generate JOIN clauses for each relation
   - Remove joins when relation filters are removed

3. **Query Assembly**
   - Combine conditions with proper operators
   - Add necessary parentheses for operator precedence
   - Append JOIN clauses when needed

## Best Practices

1. **Model Configuration**
   - Define clear relationships between models
   - Use consistent naming for foreign keys
   - Include all necessary fields in the model definition

2. **Column Setup**
   - Provide clear labels for fields
   - Include proper join information for relations
   - Set correct field types for proper operator selection

3. **Query Optimization**
   - Only include necessary joins
   - Remove joins when relation filters are deleted
   - Use appropriate indexes on join fields

## Troubleshooting

1. **Missing Joins**
   - Verify join information in column configuration
   - Check if relation filters are properly set
   - Ensure relation group has valid rules

2. **Incorrect Queries**
   - Validate rule structure
   - Check operator compatibility with field types
   - Verify condition grouping logic

3. **Performance Issues**
   - Minimize unnecessary joins
   - Index frequently filtered fields
   - Optimize query structure for large datasets

## Common Issues and Solutions

### 1. Joins Not Being Removed
If LEFT JOINs persist after deleting relation filters:
- Verify the relation group is properly cleaned
- Check that `hasValidRelationRules` is being called
- Ensure joins array is being filtered correctly

```javascript
// Check for valid relation rules
const hasValidRelationRules = (group) => {
  if (!group?.rules?.length) return false;
  return group.rules.some(rule => 
    rule.field && 
    rule.field.includes('.') && 
    rule.operator && 
    (rule.operator === 'has_records' || rule.value?.trim())
  );
};

// Filter joins based on active rules
const getRequiredJoins = (groups) => {
  const relationGroup = groups.find(g => g.isRelationship);
  if (!relationGroup?.rules?.length) return [];
  
  // Get tables used in active rules
  const usedTables = new Set(
    relationGroup.rules
      .filter(rule => rule.field?.includes('.'))
      .map(rule => rule.field.split('.')[0])
  );

  return joins.filter(join => usedTables.has(join.targetTable));
};
```

### 2. State Updates Not Reflecting
If changes to filters don't update the UI:
- Use functional updates with setState
- Deep clone objects to ensure state immutability
- Verify update handlers are properly bound

```javascript
const handleUpdateGroups = useCallback((updatedGroup, groupId) => {
  setGroups(prevGroups => {
    const newGroups = prevGroups.map(group => 
      group.id === groupId ? { ...updatedGroup[0], id: groupId } : group
    );
    
    const cleanedGroups = cleanGroups(newGroups);
    const hasRelationRules = cleanedGroups.some(
      group => group.isRelationship && group.rules.length > 0
    );
    
    const joins = hasRelationRules ? getRequiredJoins(cleanedGroups) : [];
    
    handleRuleChange({
      groups: newGroups,
      cleanedGroups,
      joins
    });
    
    return newGroups;
  });
}, [handleRuleChange]);
