---
name: learning-systems
description: 'Implement self-learning AI systems with feedback loops, performance tracking, critic evaluation, and continuous improvement patterns. Supports building agents that remember past interactions, analyze outcomes, and autonomously improve their responses over time.'
---

# Learning Systems (ระบบเรียนรู้อัตโนมัติ)

Design and implement self-improving AI systems that learn from feedback and continuously optimize their behavior.

## Core Components

### 1. Learning Element (ตัวเรียนรู้)

The learning element captures experiences and updates the agent's knowledge base:

```python
class LearningElement:
    def __init__(self):
        self.knowledge_base = {}
        self.experience_log = []

    def learn_from_feedback(self, context, action, outcome, feedback):
        """Record what worked and what didn't."""
        experience = {
            "context": context,
            "action": action,
            "outcome": outcome,
            "feedback": feedback,
            "timestamp": datetime.now().isoformat()
        }
        self.experience_log.append(experience)
        self._update_knowledge(context, action, feedback)

    def _update_knowledge(self, context, action, feedback):
        key = f"{context}:{action}"
        if key not in self.knowledge_base:
            self.knowledge_base[key] = {"successes": 0, "failures": 0}
        if feedback == "positive":
            self.knowledge_base[key]["successes"] += 1
        else:
            self.knowledge_base[key]["failures"] += 1
```

### 2. Performance Element (ตัววัดประสิทธิภาพ)

Track and evaluate agent performance over time:

```python
class PerformanceElement:
    def __init__(self):
        self.metrics = {
            "accuracy": [],
            "response_time": [],
            "user_satisfaction": []
        }

    def record_metric(self, metric_name, value):
        if metric_name in self.metrics:
            self.metrics[metric_name].append(value)

    def get_trend(self, metric_name, window=10):
        """Get recent trend for a metric."""
        recent = self.metrics.get(metric_name, [])[-window:]
        if len(recent) < 2:
            return "insufficient_data"
        return "improving" if recent[-1] > recent[0] else "declining"
```

### 3. Critic System (ระบบวิจารณ์)

Evaluate quality of responses and decisions:

```python
class CriticSystem:
    def evaluate_response(self, question, response, ground_truth=None):
        """Score a response on multiple dimensions."""
        scores = {
            "relevance": self._check_relevance(question, response),
            "completeness": self._check_completeness(response),
            "clarity": self._check_clarity(response)
        }
        return scores

    def generate_improvement_suggestion(self, response, scores):
        suggestions = []
        if scores["relevance"] < 0.7:
            suggestions.append("Address the question more directly")
        if scores["completeness"] < 0.7:
            suggestions.append("Provide more comprehensive coverage")
        if scores["clarity"] < 0.7:
            suggestions.append("Simplify language and add examples")
        return suggestions
```

### 4. Problem Generator (ตัวสร้างโจทย์)

Generate practice scenarios to expand learning:

```python
class ProblemGenerator:
    def __init__(self, domain):
        self.domain = domain
        self.difficulty_levels = ["beginner", "intermediate", "advanced"]

    def generate_scenario(self, difficulty="intermediate", topic=None):
        """Create a new learning scenario."""
        return {
            "domain": self.domain,
            "difficulty": difficulty,
            "topic": topic or self._select_topic(),
            "objective": self._create_objective(difficulty),
            "evaluation_criteria": self._define_criteria()
        }
```

## Implementation Patterns

### Feedback Loop Architecture

```
User Input
    ↓
Agent Response
    ↓
Critic Evaluation ──→ Performance Metrics
    ↓
Learning Update
    ↓
Improved Next Response
```

### Memory Management

Store and retrieve relevant past experiences efficiently:

```python
def retrieve_relevant_experience(self, current_context, top_k=5):
    """Find most similar past experiences."""
    scored = [
        (exp, self._similarity(current_context, exp["context"]))
        for exp in self.experience_log
    ]
    return sorted(scored, key=lambda x: x[1], reverse=True)[:top_k]
```

## Usage Guidelines

1. **Start simple** - Begin with basic positive/negative feedback before complex evaluations
2. **Log everything** - Comprehensive logs enable better learning
3. **Set clear metrics** - Define what "good performance" means before building
4. **Test feedback loops** - Verify the system actually improves with positive feedback
5. **Guard against overfitting** - Balance exploitation of known patterns with exploration of new ones
