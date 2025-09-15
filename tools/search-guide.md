# Problem Search Guide 🔍

## GitHub Search Syntax

Use GitHub's powerful search to find relevant solutions:

### Basic Search
```
error message text
```

### Search by File Type
```
filename:*.md database connection
```

### Search by Path
```
path:problems/database/ slow query
```

### Search by Date
```
created:>2024-01-01 authentication
```

### Combination Search
```
path:problems/ "error 500" created:>2024-06-01
```

## Search Strategies

### 1. Start with Error Messages
- Copy exact error text
- Remove variable parts (IDs, timestamps)
- Search for the core error message

### 2. Search by Technology
- Include framework names (React, Django, etc.)
- Include database names (PostgreSQL, MongoDB)
- Include service names (AWS, Docker)

### 3. Search by Symptoms
- "slow performance"
- "connection timeout"
- "build failing"

### 4. Browse by Category
- Check the appropriate folder first
- Review the INDEX.md for similar issues

## Quick Reference Commands

### Find all critical issues
```
severity-critical in:file
```

### Find recent solutions
```
path:problems/ created:>2024-09-01
```

### Find quick fixes
```
"quick-fix" in:file
```

## Pro Tips

1. **Use quotes** for exact phrases
2. **Remove timestamps** from error messages  
3. **Search incrementally** - start broad, then narrow
4. **Check related issues** in the same category
5. **Use the INDEX.md** as a starting point