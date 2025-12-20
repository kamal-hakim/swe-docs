# Ticket: BE-006 - Create Task Command (CQRS Write Side)

**Epic**: Project & Task Domain
**Type**: Story
**Priority**: High
**Dependencies**: BE-005

## 📝 Story
As a user, I want to add a task to a project so that I can track work items.

## ✅ Acceptance Criteria
- [ ] `CreateTaskInteractor` implemented.
- [ ] `TaskRepository` interface defined.
- [ ] Invariant check: User must be a member of the project (Mock logic for now).
- [ ] Repository implementation.
- [ ] API Endpoint `POST /api/v1/projects/{id}/tasks`.

## 🛠️ Solution Approach

### CQRS Command
This is a **Write** operation.
1. **Input**: `CreateTaskRequest(title, description, priority)`.
2. **Logic**:
   - Load Project.
   - Check if Current User has write access.
   - Create `Task` entity.
   - Save.

### Best Practices Explained
> **Transactional Boundary**: The Interactor defines the transaction. All repository operations (add task, maybe update project stats) must happen within one `uow.commit()`.

```python
async def execute(self, request):
    # Authorization checks first
    # ...
    task = Task(...)
    repo.add(task)
    await uow.commit()
```
