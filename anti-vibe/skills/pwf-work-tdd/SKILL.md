---
name: pwf-work-tdd
description: >
  USE WHEN: Explicitly requested tests-first behavior. Implements features using
  red-green-refactor cycle with test-first discipline.
  
  DON'T USE WHEN: Tests-first not explicitly requested (use pwf-work or
  pwf-work-plan instead). This skill requires explicit opt-in.
  
  REQUIRED INPUT: Feature description with clear expected behavior.
  
  OUTPUT: Implementation with test coverage, following TDD cycle.
  
  PROCESS:
  1. Write failing test (red)
  2. Implement minimal code to pass (green)
  3. Refactor while keeping tests passing
  4. Repeat until feature complete
  
  CYCLE: Red → Green → Refactor
---

# Test-Driven Development

Use this skill when explicitly requested to use tests-first behavior.

**Opt-in only** — requires explicit user request for TDD.

---

## Paperclip Integration

TDD cycles align well with Paperclip heartbeat model:
- Each red-green-refactor cycle can complete within a heartbeat
- Test results provide clear verification evidence
- Task progress updates via API between cycles

---

## TDD Cycle

### Phase 1: Red (Write Failing Test)

1. **Understand requirement** from feature description
2. **Write test** that defines expected behavior
3. **Run test** — confirm it fails (red)
4. **Commit test** (optional but recommended)

Rules:
- Test must fail for right reason (not compilation error)
- Test clearly describes expected behavior
- Test is minimal — one concept per test

### Phase 2: Green (Make Test Pass)

1. **Implement minimal code** to make test pass
2. **Run test** — confirm it passes (green)
3. **No refactoring yet** — just make it work

Rules:
- Minimal implementation — "cheating" allowed if test passes
- Don't worry about code quality yet
- Focus: get to green as fast as possible

### Phase 3: Refactor (Clean Code)

1. **Improve code** while keeping tests green
2. **Run all tests** — ensure no regressions
3. **Commit** refactoring changes

Rules:
- Tests must stay green throughout
- Refactor one thing at a time
- Run tests after each small change

### Phase 4: Repeat

Continue cycle until feature complete:
- Next requirement → new test (red)
- Make it pass (green)
- Refactor (clean)

---

## Full Workflow Integration

### Step 1: Research (Minimal)

- Read docs for the feature scope
- Understand existing test patterns
- Identify test utilities/helpers

### Step 2: TDD Cycles

Execute red-green-refactor cycles until feature complete.

### Step 3: Validation

```bash
npm run validate       # TypeScript + lint
npm test              # All tests
```

### Step 4: Documentation

Apply `docs-maintenance-after-work` skill:
- Update module/feature docs
- Document test patterns used
- Capture any learnings

### Step 5: Review

If >5 files touched or complex logic:
- Run `pwf-review`
- Ensure test coverage is adequate

---

## Test Patterns by Layer

### Backend (NestJS)

```typescript
// Service test pattern
describe('FeatureService', () => {
  let service: FeatureService;
  let repository: MockType<Repository<Feature>>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        FeatureService,
        { provide: getRepositoryToken(Feature), useFactory: repositoryMockFactory }
      ]
    }).compile();

    service = module.get(FeatureService);
    repository = module.get(getRepositoryToken(Feature));
  });

  it('should [expected behavior]', () => {
    // Arrange
    const input = ...;
    const expected = ...;

    // Act
    const result = service.method(input);

    // Assert
    expect(result).toEqual(expected);
  });
});
```

### Frontend (Angular)

```typescript
// Component test pattern
describe('FeatureComponent', () => {
  let component: FeatureComponent;
  let fixture: ComponentFixture<FeatureComponent>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [FeatureComponent],
      providers: [{ provide: FeatureService, useValue: mockService }]
    });

    fixture = TestBed.createComponent(FeatureComponent);
    component = fixture.componentInstance;
  });

  it('should [expected behavior]', () => {
    // Test implementation
  });
});
```

---

## When to STOP TDD

Stop TDD cycle and reassess if:
- Tests become too complex (test code > implementation code)
- Multiple cycles without progress (stuck on design)
- Feature scope was misunderstood

Options:
- Switch to `pwf-work` for exploratory implementation
- Run `pwf-brainstorm` to clarify approach
- Break feature into smaller pieces

---

## Verification Evidence

Provide:
- Test count: X tests written
- Coverage: [if available]
- All tests passing: ✅
- TypeScript validation: ✅

---

## Conventions

- One test, one concept
- Test behavior, not implementation
- Tests are documentation — make them readable
- Refactor aggressively in green phase

## Next Recommended Skills

- `pwf-review` — after TDD completion
- `pwf-commit-changes` — structured commits
- `pwf-doc-capture` — document patterns discovered
