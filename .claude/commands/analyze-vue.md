---
description: Analyze Vue component structure and suggest optimizations for performance and code reuse
---

# Vue Component Analysis

Perform a comprehensive analysis of all Vue components in `client/src/`. Use the **Explore** agent to scan the codebase, then provide a structured report.

## Step 1: Component Inventory

Read every `.vue` file in `client/src/views/` and `client/src/components/`. For each component, extract:

- **Component name** and file path
- **Template size** (approximate line count for `<template>`, `<script>`, `<style>` sections)
- **Props** received (if any)
- **Emits** declared (if any)
- **Composables / imports** used
- **Reactive state** (refs, reactive, computed, watchers)
- **API calls** made (via `api.js` or direct)
- **Child components** used in the template

Present this as a summary table.

## Step 2: Performance Analysis

Check each component for these performance issues:

### Reactivity
- **Unnecessary watchers**: watchers that could be replaced with computed properties
- **Missing `shallowRef`**: large arrays/objects stored in `ref()` that don't need deep reactivity
- **Expensive computed properties**: computed properties that do heavy filtering/sorting on every dependency change without memoization
- **Inline functions in templates**: arrow functions in `v-on` or `:prop` bindings that create new references each render

### Rendering
- **Large `v-for` lists** without virtual scrolling or pagination
- **Missing `:key` or using index as key** in `v-for` loops
- **Unnecessary re-renders**: components that re-render when parent state changes but their props haven't changed (candidates for `v-memo` or component extraction)
- **Heavy inline template logic**: complex expressions in templates that should be computed properties

### Data Loading
- **Redundant API calls**: multiple components fetching the same data independently
- **Missing loading/error states**: API calls without proper loading indicators or error handling
- **No request deduplication**: same endpoint called multiple times on filter changes

## Step 3: Code Reuse Analysis

Identify opportunities to reduce duplication:

### Shared Patterns
- **Repeated template patterns**: similar table structures, card layouts, chart containers, modal patterns across components
- **Duplicated filter logic**: filter handling code that appears in multiple views
- **Common formatting functions**: number formatting, date formatting, status badge logic used in multiple places
- **Repeated API patterns**: similar fetch-filter-display patterns across views

### Extraction Candidates
For each duplication found, suggest:
1. **What to extract**: the specific code pattern
2. **Extract as what**: composable function, base component, utility function, or directive
3. **Which files benefit**: list of files that would use the extraction
4. **Estimated line reduction**: approximate lines of code saved

### Composable Opportunities
Look for stateful logic that should be extracted into `composables/`:
- Filter state management (shared across views)
- Data fetching + loading state patterns
- Table sorting/pagination logic
- Modal open/close state management
- Number/date formatting utilities

## Step 4: Output Report

Organize findings into three priority levels:

### Critical (fix now)
- Performance issues that affect user experience (large unvirtualized lists, redundant API calls, missing keys)
- Bugs or anti-patterns that will cause issues as the app grows

### Recommended (improve soon)
- Code reuse opportunities that would reduce maintenance burden by >20 lines
- Composable extractions for logic shared by 3+ components
- Template simplifications that improve readability

### Nice-to-have (consider later)
- Minor optimizations with small impact
- Style consolidation
- Naming consistency improvements

For each finding, include:
- **File(s)** affected with line numbers
- **Current code** snippet showing the issue
- **Suggested fix** with example code
- **Impact**: what improves (bundle size, render performance, maintainability, DX)

Do NOT make any changes to files. This is an analysis-only command. Present findings clearly so the user can decide what to act on.
