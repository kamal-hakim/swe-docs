# Ticket: BE-005 - Project and Task Domain Entities

**Epic**: Project & Task Domain
**Type**: Story
**Priority**: High
**Dependencies**: BE-003

## 📝 Story
As a domain expert, I need the core business entities `Project` and `Task` defined so that we can model the project management workflow.

## ✅ Acceptance Criteria
- [ ] `Project` entity defined with `id`, `name`, `owner_id`.
- [ ] `Task` entity defined with `id`, `project_id`, `title`, `priority` (Enum).
- [ ] `Task` invariants:
    - Priority defaults to MEDIUM.
    - Tasks must belong to a project.
    - Creator must be specified.
- [ ] Unit tests for `Task` priority logic.

## 🛠️ Solution Approach

### Entities
```python
class Project(Entity):
    def __init__(self, name: str, owner_id: UserId):
        self.name = name
        self.owner_id = owner_id

class Task(Entity):
    def __init__(self, title: str, project_id: ProjectId, creator_id: UserId):
        self.priority = Priority.MEDIUM
        # ...
```

### Best Practices Explained
> **Rich Model**: Over time we will add methods like `convert_to_subtask()` or `change_priority()` here.
> **Enums**: Use Python `Enum` for Priority (`LOW`, `MEDIUM`, `HIGH`) to ensure type safety.

## 🔗 References
- [Domain Layer](../docs/backend/03-domain-layer.md)
