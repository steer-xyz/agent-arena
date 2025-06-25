# Agentic Arena - Evaluation Metrics Specification

## Overview
This document defines the comprehensive evaluation system for ranking and scoring workflow submissions across different AI coding tools. The evaluation framework considers efficiency, speed, cost, and correctness to provide fair and meaningful comparisons.

## Core Evaluation Principles

### 1. Multi-Dimensional Scoring
- **No single metric dominates**: Balanced scoring across multiple dimensions
- **Context-aware evaluation**: Different tasks may weight metrics differently
- **Normalization**: Scores normalized to 0-100 scale for consistency
- **Transparency**: All scoring algorithms are open and auditable

### 2. Fairness Across Tools
- **Tool-agnostic metrics**: Focus on outcomes rather than tool-specific features
- **Comparable baselines**: Establish baselines for each tool/model combination
- **Statistical significance**: Require minimum submissions for reliable rankings
- **Bias detection**: Monitor for systematic advantages/disadvantages

## Primary Evaluation Metrics

### 1. Efficiency Score (35% weight)
Measures how efficiently the workflow uses computational resources.

#### Token Efficiency
```python
def calculate_token_efficiency_score(submission: dict, task_baseline: dict) -> float:
    """
    Calculate token efficiency score based on total tokens used.
    Lower token usage = higher score.
    """
    tokens_used = submission['total_tokens']
    baseline_tokens = task_baseline['median_tokens']
    min_tokens = task_baseline['min_tokens']
    max_tokens = task_baseline['max_tokens']
    
    if tokens_used <= min_tokens:
        return 100.0
    elif tokens_used >= max_tokens:
        return 0.0
    else:
        # Logarithmic scoring favors significant efficiency gains
        efficiency_ratio = (max_tokens - tokens_used) / (max_tokens - min_tokens)
        return min(100.0, efficiency_ratio * 100.0)
```

#### Tool Call Efficiency
```python
def calculate_tool_call_efficiency_score(submission: dict, task_baseline: dict) -> float:
    """
    Calculate efficiency based on number of tool/function calls.
    Fewer calls generally indicate more efficient workflow.
    """
    tool_calls = submission['tool_calls']
    baseline_calls = task_baseline['median_tool_calls']
    min_calls = task_baseline['min_tool_calls']
    max_calls = task_baseline['max_tool_calls']
    
    if tool_calls <= min_calls:
        return 100.0
    elif tool_calls >= max_calls:
        return 0.0
    else:
        efficiency_ratio = (max_calls - tool_calls) / (max_calls - min_calls)
        return efficiency_ratio * 100.0
```

#### Iteration Efficiency
```python
def calculate_iteration_efficiency_score(submission: dict, task_baseline: dict) -> float:
    """
    Calculate efficiency based on number of iterations/refinements.
    Fewer iterations indicate better initial approach.
    """
    iterations = submission.get('iterations', 1)
    baseline_iterations = task_baseline['median_iterations']
    
    if iterations == 1:
        return 100.0
    elif iterations <= baseline_iterations:
        return max(50.0, 100.0 - (iterations - 1) * 15.0)
    else:
        return max(0.0, 50.0 - (iterations - baseline_iterations) * 10.0)
```

#### Combined Efficiency Score
```python
def calculate_efficiency_score(submission: dict, task_baseline: dict) -> float:
    """Combine all efficiency metrics with weights."""
    token_score = calculate_token_efficiency_score(submission, task_baseline)
    tool_call_score = calculate_tool_call_efficiency_score(submission, task_baseline)
    iteration_score = calculate_iteration_efficiency_score(submission, task_baseline)
    
    # Weighted combination
    return (
        token_score * 0.5 +
        tool_call_score * 0.3 + 
        iteration_score * 0.2
    )
```

### 2. Speed Score (25% weight)
Measures how quickly the workflow was completed.

```python
def calculate_speed_score(submission: dict, task_baseline: dict) -> float:
    """
    Calculate speed score based on execution time.
    Faster completion = higher score.
    """
    execution_time = submission.get('execution_time', 0)
    if execution_time == 0:
        return 50.0  # Default score for missing timing data
    
    baseline_time = task_baseline['median_execution_time']
    min_time = task_baseline['min_execution_time']
    max_time = task_baseline['max_execution_time']
    
    if execution_time <= min_time:
        return 100.0
    elif execution_time >= max_time:
        return 0.0
    else:
        # Exponential scoring to reward significant speed improvements
        speed_ratio = (max_time - execution_time) / (max_time - min_time)
        return min(100.0, (speed_ratio ** 0.7) * 100.0)
```

### 3. Cost Score (20% weight)
Measures the estimated monetary cost of the workflow.

```python
def calculate_cost_score(submission: dict, task_baseline: dict) -> float:
    """
    Calculate cost efficiency score based on estimated API costs.
    Lower cost = higher score.
    """
    estimated_cost = submission.get('estimated_cost', 0.0)
    if estimated_cost == 0.0:
        return 50.0  # Default score for missing cost data
    
    baseline_cost = task_baseline['median_cost']
    min_cost = task_baseline['min_cost']
    max_cost = task_baseline['max_cost']
    
    if estimated_cost <= min_cost:
        return 100.0
    elif estimated_cost >= max_cost:
        return 0.0
    else:
        cost_ratio = (max_cost - estimated_cost) / (max_cost - min_cost)
        return cost_ratio * 100.0
```

### 4. Correctness Score (20% weight)
Measures the quality and correctness of the final output.

```python
def calculate_correctness_score(submission: dict, task: dict) -> float:
    """
    Calculate correctness score based on task-specific criteria.
    This requires human evaluation or automated testing.
    """
    # This would be implemented per task type
    # For now, we'll use a placeholder that considers basic success criteria
    
    success_criteria = task.get('success_criteria', [])
    if not success_criteria:
        return 100.0  # Default to full score if no criteria defined
    
    # Check each success criterion
    passed_criteria = []
    for criterion in success_criteria:
        # This would involve actual testing/validation
        # For now, assume all successful submissions meet basic criteria
        passed_criteria.append(check_criterion(submission, criterion))
    
    if not success_criteria:
        return 100.0
    
    pass_rate = sum(passed_criteria) / len(success_criteria)
    return pass_rate * 100.0

def check_criterion(submission: dict, criterion: dict) -> bool:
    """Check if submission meets a specific success criterion."""
    # Implementation depends on criterion type
    # Examples: file_exists, code_compiles, tests_pass, etc.
    criterion_type = criterion.get('type')
    
    if criterion_type == 'file_exists':
        # Check if required files were created
        return criterion['file_path'] in submission.get('files_created', [])
    
    elif criterion_type == 'no_errors':
        # Check if workflow completed without errors
        return submission.get('success', True)
    
    elif criterion_type == 'min_functionality':
        # Basic functionality check (would need automated testing)
        return True  # Placeholder
    
    return True  # Default to passing
```

## Overall Score Calculation

### Weighted Composite Score
```python
def calculate_overall_score(submission: dict, task: dict, task_baseline: dict) -> dict:
    """Calculate the overall weighted score for a submission."""
    
    # Calculate individual scores
    efficiency_score = calculate_efficiency_score(submission, task_baseline)
    speed_score = calculate_speed_score(submission, task_baseline)
    cost_score = calculate_cost_score(submission, task_baseline)
    correctness_score = calculate_correctness_score(submission, task)
    
    # Apply weights
    weights = {
        'efficiency': 0.35,
        'speed': 0.25,
        'cost': 0.20,
        'correctness': 0.20
    }
    
    overall_score = (
        efficiency_score * weights['efficiency'] +
        speed_score * weights['speed'] +
        cost_score * weights['cost'] +
        correctness_score * weights['correctness']
    )
    
    return {
        'overall_score': round(overall_score, 2),
        'efficiency_score': round(efficiency_score, 2),
        'speed_score': round(speed_score, 2),
        'cost_score': round(cost_score, 2),
        'correctness_score': round(correctness_score, 2),
        'breakdown': {
            'efficiency_weight': weights['efficiency'],
            'speed_weight': weights['speed'],
            'cost_weight': weights['cost'],
            'correctness_weight': weights['correctness']
        }
    }
```

## Baseline Calculation

### Dynamic Baseline Updates
```python
def calculate_task_baseline(task_id: str, submissions: list) -> dict:
    """
    Calculate baseline metrics for a task based on all valid submissions.
    Baselines are updated as new submissions arrive.
    """
    if len(submissions) < 5:
        # Not enough data for reliable baseline
        return get_default_baseline(task_id)
    
    # Extract metrics from all submissions
    tokens = [s['total_tokens'] for s in submissions]
    tool_calls = [s['tool_calls'] for s in submissions]
    execution_times = [s.get('execution_time', 0) for s in submissions if s.get('execution_time', 0) > 0]
    costs = [s.get('estimated_cost', 0) for s in submissions if s.get('estimated_cost', 0) > 0]
    iterations = [s.get('iterations', 1) for s in submissions]
    
    return {
        'submission_count': len(submissions),
        'median_tokens': statistics.median(tokens),
        'min_tokens': min(tokens),
        'max_tokens': max(tokens),
        'q25_tokens': statistics.quantiles(tokens, n=4)[0],
        'q75_tokens': statistics.quantiles(tokens, n=4)[2],
        
        'median_tool_calls': statistics.median(tool_calls),
        'min_tool_calls': min(tool_calls),
        'max_tool_calls': max(tool_calls),
        
        'median_execution_time': statistics.median(execution_times) if execution_times else 300,
        'min_execution_time': min(execution_times) if execution_times else 60,
        'max_execution_time': max(execution_times) if execution_times else 1800,
        
        'median_cost': statistics.median(costs) if costs else 0.05,
        'min_cost': min(costs) if costs else 0.01,
        'max_cost': max(costs) if costs else 0.20,
        
        'median_iterations': statistics.median(iterations),
        'last_updated': datetime.utcnow().isoformat()
    }
```

## Ranking System

### Overall Rankings
```python
def calculate_rankings(evaluations: list) -> list:
    """
    Calculate rankings for all submissions to a task.
    Handle ties appropriately.
    """
    # Sort by overall score (descending)
    sorted_evaluations = sorted(
        evaluations, 
        key=lambda x: (x['overall_score'], x['evaluated_at']), 
        reverse=True
    )
    
    # Assign rankings, handling ties
    rankings = []
    current_rank = 1
    
    for i, evaluation in enumerate(sorted_evaluations):
        if i > 0 and evaluation['overall_score'] == sorted_evaluations[i-1]['overall_score']:
            # Tie - use same rank
            ranking = rankings[-1]['overall_ranking']
        else:
            ranking = current_rank
        
        rankings.append({
            **evaluation,
            'overall_ranking': ranking,
            'percentile': calculate_percentile(evaluation['overall_score'], [e['overall_score'] for e in evaluations])
        })
        
        current_rank = i + 2  # Next unique rank
    
    return rankings
```

### Category-Specific Rankings
```python
def calculate_category_rankings(evaluations: list, metric: str) -> list:
    """Calculate rankings for a specific metric (efficiency, speed, cost)."""
    metric_key = f'{metric}_score'
    
    sorted_evaluations = sorted(
        evaluations,
        key=lambda x: x[metric_key],
        reverse=True
    )
    
    for i, evaluation in enumerate(sorted_evaluations):
        evaluation[f'{metric}_ranking'] = i + 1
    
    return sorted_evaluations
```

## Statistical Analysis

### Percentile Calculation
```python
def calculate_percentile(score: float, all_scores: list) -> float:
    """Calculate what percentile a score represents."""
    if not all_scores:
        return 50.0
    
    sorted_scores = sorted(all_scores)
    position = sorted_scores.index(score)
    percentile = (position / len(sorted_scores)) * 100
    return round(percentile, 1)
```

### Confidence Intervals
```python
def calculate_confidence_interval(scores: list, confidence_level: float = 0.95) -> tuple:
    """Calculate confidence interval for score distribution."""
    if len(scores) < 3:
        return (0, 100)  # Not enough data
    
    mean = statistics.mean(scores)
    std_dev = statistics.stdev(scores)
    n = len(scores)
    
    # t-distribution for small samples
    if n < 30:
        t_value = 2.776  # t-value for 95% confidence with small samples
    else:
        t_value = 1.96   # z-value for 95% confidence with large samples
    
    margin_of_error = t_value * (std_dev / math.sqrt(n))
    
    return (
        max(0, mean - margin_of_error),
        min(100, mean + margin_of_error)
    )
```

## Task-Specific Customizations

### Task Categories and Weights
```python
TASK_CATEGORY_WEIGHTS = {
    'frontend_development': {
        'efficiency': 0.30,
        'speed': 0.25,
        'cost': 0.15,
        'correctness': 0.30  # Higher weight on correctness for UI
    },
    'backend_development': {
        'efficiency': 0.35,
        'speed': 0.30,
        'cost': 0.20,
        'correctness': 0.15
    },
    'data_analysis': {
        'efficiency': 0.40,
        'speed': 0.20,
        'cost': 0.25,
        'correctness': 0.15
    },
    'debugging': {
        'efficiency': 0.25,
        'speed': 0.35,  # Speed is crucial for debugging
        'cost': 0.15,
        'correctness': 0.25
    },
    'refactoring': {
        'efficiency': 0.45,  # Efficiency is key for refactoring
        'speed': 0.20,
        'cost': 0.20,
        'correctness': 0.15
    }
}
```

### Difficulty Adjustments
```python
def apply_difficulty_adjustment(score: float, difficulty: str) -> float:
    """Adjust scores based on task difficulty."""
    adjustments = {
        'beginner': 1.0,     # No adjustment
        'intermediate': 1.1,  # 10% bonus
        'advanced': 1.2      # 20% bonus
    }
    
    return min(100.0, score * adjustments.get(difficulty, 1.0))
```

## Quality Assurance

### Outlier Detection
```python
def detect_outliers(submissions: list) -> list:
    """Detect and flag potentially suspicious submissions."""
    outliers = []
    
    # Check for extreme efficiency (too good to be true)
    tokens = [s['total_tokens'] for s in submissions]
    token_threshold = statistics.median(tokens) * 0.1  # 10% of median
    
    for submission in submissions:
        flags = []
        
        if submission['total_tokens'] < token_threshold:
            flags.append('extremely_efficient_tokens')
        
        if submission.get('execution_time', 0) < 5:  # Less than 5 seconds
            flags.append('extremely_fast')
        
        if submission.get('tool_calls', 0) == 1:  # Single tool call
            flags.append('minimal_tool_usage')
        
        if len(flags) >= 2:  # Multiple suspicious indicators
            outliers.append({
                'submission_id': submission['id'],
                'flags': flags,
                'recommended_action': 'manual_review'
            })
    
    return outliers
```

### Validation Rules
```python
def validate_evaluation_metrics(evaluation: dict) -> list:
    """Validate that evaluation metrics are reasonable."""
    errors = []
    
    # Check score ranges
    for score_type in ['overall', 'efficiency', 'speed', 'cost', 'correctness']:
        score_key = f'{score_type}_score'
        if score_key in evaluation:
            score = evaluation[score_key]
            if not (0 <= score <= 100):
                errors.append(f'{score_type}_score out of range: {score}')
    
    # Check for impossible combinations
    if evaluation.get('efficiency_score', 0) > 95 and evaluation.get('speed_score', 0) > 95:
        errors.append('Suspicious: Both efficiency and speed scores are extremely high')
    
    # Check token consistency
    if evaluation.get('total_tokens', 0) < 10:
        errors.append('Suspiciously low token count')
    
    return errors
```

## Evaluation Pipeline

### Processing Workflow
1. **Submission Parsing**: Extract metrics from workflow data
2. **Baseline Retrieval**: Get current baselines for the task
3. **Score Calculation**: Calculate all metric scores
4. **Outlier Detection**: Flag suspicious submissions
5. **Ranking Update**: Update rankings with new submission
6. **Baseline Update**: Recalculate baselines if needed
7. **Notification**: Notify user of evaluation completion

### Batch Processing
```python
async def process_evaluation_batch(submissions: list) -> list:
    """Process multiple evaluations efficiently."""
    results = []
    
    # Group by task for batch baseline updates
    by_task = {}
    for submission in submissions:
        task_id = submission['task_id']
        if task_id not in by_task:
            by_task[task_id] = []
        by_task[task_id].append(submission)
    
    # Process each task group
    for task_id, task_submissions in by_task.items():
        task = await get_task(task_id)
        baseline = await get_task_baseline(task_id)
        
        for submission in task_submissions:
            evaluation = calculate_overall_score(submission, task, baseline)
            evaluation['submission_id'] = submission['id']
            evaluation['task_id'] = task_id
            evaluation['evaluated_at'] = datetime.utcnow().isoformat()
            
            results.append(evaluation)
        
        # Update baseline with new submissions
        await update_task_baseline(task_id, task_submissions)
    
    return results
```

## Future Enhancements

### Planned Improvements
1. **ML-Based Scoring**: Use machine learning to improve correctness evaluation
2. **Peer Review Integration**: Incorporate community ratings into scoring
3. **Tool-Specific Metrics**: Add metrics specific to certain AI tools
4. **Historical Trending**: Track score evolution over time
5. **A/B Testing Framework**: Test different scoring algorithms
6. **Real-time Adjustments**: Dynamic weight adjustments based on community feedback
7. **Cross-Task Learning**: Use performance patterns across similar tasks
8. **Advanced Statistics**: More sophisticated statistical analysis and confidence measures