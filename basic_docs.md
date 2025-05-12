# Guidelines: Learning from API Hallucination Errors

## Executive Summary
This document outlines the failures encountered while attempting to help with the OpenAI Agents SDK error handling, the root causes of these failures, and guidelines to prevent similar issues in the future.

## The Problem Timeline

### Initial Failure: Import Statement
**What I suggested:** 
```python
from agents.exceptions import InputGuardrailTripwireTriggered
```

**Why it failed:** I assumed the exception would be in a submodule based on common Python package patterns, but the OpenAI Agents SDK exposes exceptions directly from the main module.

**Correct solution:**
```python
from agents import InputGuardrailTripwireTriggered
```

### Second Failure: Exception Structure
**What I suggested:**
```python
guardrail_output = e.guardrail_result.output_info
```

**Why it failed:** I assumed the exception structure without verifying it against the actual API. The real structure has an additional layer: `e.guardrail_result.output.output_info`

**Correct solution:**
```python
guardrail_output = e.guardrail_result.output.output_info
```

## Root Causes of Failures

### 1. Pattern Matching Assumptions
- **Error:** Applied common Python patterns without verification
- **Reality:** Each SDK has its own conventions; OpenAI Agents SDK keeps imports flat
- **Lesson:** Never assume package structure - always verify with documentation or testing

### 2. API Structure Hallucination  
- **Error:** Guessed at object structure based on variable names
- **Reality:** The actual structure had nested objects with specific attribute names
- **Lesson:** When unsure about API structure, suggest debugging approaches first

### 3. Documentation Misinterpretation
- **Error:** Extrapolated beyond what documentation showed
- **Reality:** Documentation examples were intentionally simple and didn't show data access
- **Lesson:** Stick to what's explicitly shown in documentation; don't invent features

## Guidelines for Preventing Hallucination

### 1. Documentation-First Approach
- ✅ Use exact code patterns from official documentation
- ✅ When documentation is silent, explicitly state uncertainty
- ❌ Don't extrapolate features that aren't documented
- ❌ Don't assume standard patterns apply to all SDKs

### 2. Debugging Before Assuming
When structure is unknown:
```python
# First, explore what's available
print(f"Exception type: {type(e)}")
print(f"Exception attributes: {dir(e)}")
print(f"Exception string: {str(e)}")

# Then drill down into nested objects
if hasattr(e, 'guardrail_result'):
    print(f"Guardrail result: {e.guardrail_result}")
    print(f"Guardrail result attributes: {dir(e.guardrail_result)}")
```

### 3. Progressive Problem Solving
1. Start with simplest solution (basic exception handling)
2. Add debugging to understand structure
3. Only then suggest specific attribute access
4. Validate each step before proceeding

### 4. Explicit Uncertainty Communication
Instead of: "The exception contains the reasoning at `e.guardrail_result.output_info`"

Better: "The exception likely contains the reasoning, but let's explore its structure first to find the correct path"

## Specific Lessons for SDK Integration

### 1. Import Patterns Vary
- Some SDKs use deep module structure: `from sdk.submodule.exceptions import Error`
- Others keep it flat: `from sdk import Error`
- Always check documentation for import examples

### 2. Exception Objects Are Complex
- Exception objects often have nested structures
- Attribute names may not be intuitive
- Always use debugging to explore structure

### 3. Framework Constraints Matter
- Stay within the framework's patterns
- Don't suggest external solutions when framework provides them
- Respect the framework's error handling philosophy

## Internal Error Analysis

The core internal error was **premature pattern matching** - applying familiar patterns from other SDKs without verification. This manifested as:

1. **Cognitive bias:** Expected patterns from popular frameworks (like Django, Flask)
2. **Overconfidence:** Provided specific solutions without verification
3. **Insufficient exploration:** Jumped to solutions instead of investigation

## Prevention Strategies

### 1. Always Start with Investigation
```python
# Before suggesting any attribute access
try:
    # ... code that might fail
except Exception as e:
    print(f"Exception details: {type(e)}, {dir(e)}")
    # Then explore based on what's available
```

### 2. Use Documentation Examples Literally
- Copy exact patterns from docs
- Don't modify without testing
- Acknowledge when going beyond docs

### 3. Fail Gracefully
- Provide fallback solutions
- Suggest debugging approaches
- Admit uncertainty when appropriate

## Conclusion

The key learning is that **hallucination occurs when pattern matching overrides verification**. Future interactions should:

1. Prioritize official documentation
2. Use debugging before assuming
3. Communicate uncertainty clearly
4. Stay within framework constraints
5. Validate each step progressively

By following these guidelines, we can provide accurate, helpful assistance without creating frustration through incorrect assumptions.
