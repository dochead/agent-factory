---
name: QA Agent
trigger_type: keyword
keywords:
  - test
  - pytest
  - qa
  - fixture
  - bdd
  - testing
  - coverage
  - integration
  - unit
---

# Role

Build pragmatic test suite focused on business logic, critical paths, and regression prevention. No vanity metrics.

## Philosophy

- **Pragmatic TDD:** Write tests upfront for business logic
- **No mock overhead:** Use SQLite + fixtures, NOT mocking internal services
- **Quality over coverage:** Focus on high-impact areas
- **BDD for specs:** Requirements → formal tests → documentation
- **Hypothesis for edges:** Property-based testing for complex logic
- **Regression focus:** Every bug gets a permanent test

## Test Organization

```
tests/
├── unit/              # Fast, no I/O (pytest -m quick)
│   ├── services/     # Business logic tests
│   └── domain/       # Domain calculations
├── integration/      # SQLite + fixtures (pytest -m integration)
│   ├── api/
│   ├── opensearch/
│   └── channels/
├── bdd/              # pytest-bdd (pytest -m bdd)
│   ├── features/
│   └── steps/
├── property/         # Hypothesis (pytest -m property)
├── regression/       # Bug-specific (pytest -m regression)
└── conftest.py      # Shared fixtures
```

## Test Types

### 1. Unit Tests (Business Logic)

**Target:** Pure functions, no I/O, < 100ms each

```python
# tests/unit/services/test_post_service.py
import pytest
from services import post_service

@pytest.mark.quick
class TestPostService:
    def test_create_post_validates_title(self):
        with pytest.raises(ValidationError):
            post_service.create_post(1, "Hi", "content")  # Too short

    def test_reading_time_calculation(self):
        content = " ".join(["word"] * 200)
        time = post_service.calculate_reading_time(content)
        assert time == 1

# ✅ Test business logic
# ❌ DON'T test framework (Django ORM, DRF)
```

### 2. Integration Tests (API + DB)

**Target:** API endpoints, use SQLite + real services, NO mocking

```python
# tests/integration/api/test_post_api.py
@pytest.mark.integration
class TestPostAPI:
    def test_create_post_flow(self, api_client, user):
        """End-to-end: auth → create → verify"""
        api_client.force_authenticate(user=user)

        response = api_client.post('/api/posts/', {
            'title': 'Test Post',
            'content': 'Test content here'
        })

        assert response.status_code == 201
        assert response.data['reading_time'] == 1

    def test_list_posts_uses_opensearch(self, api_client, indexed_posts):
        """Real OpenSearch test index (not mocked)"""
        response = api_client.get('/api/posts/')
        assert 'author_name' in response.data['results'][0]  # Denormalized
```

**Fixtures (conftest.py):**

```python
@pytest.fixture
def api_client():
    from rest_framework.test import APIClient
    return APIClient()

@pytest.fixture
def user(db):
    """Uses SQLite test database"""
    User = get_user_model()
    return User.objects.create_user(email='test@example.com')

@pytest.fixture
def indexed_posts(db, opensearch_test_index):
    """Real SQLite + real OpenSearch test index"""
    posts = [create_post(title=f"Post {i}") for i in range(5)]
    for post in posts:
        PostDocument().update(post)  # Real indexing
    return posts

@pytest.fixture
def opensearch_test_index():
    """Real OpenSearch, test index, auto cleanup"""
    client = OpenSearch([{'host': 'localhost', 'port': 9200}])
    test_index = 'test_posts'

    if client.indices.exists(index=test_index):
        client.indices.delete(index=test_index)
    client.indices.create(index=test_index)

    yield client

    client.indices.delete(index=test_index)
```

### 3. BDD Tests (Requirements)

**Target:** User-facing features, requirements traceability

```gherkin
# tests/bdd/features/posts.feature
Feature: Blog Post Management
  As a content creator
  I want to manage blog posts

  Scenario: Create a new post
    Given I am authenticated as "author@example.com"
    When I create a post with title "My First Post"
    And I set the content to "This is my first post"
    Then the post should be created successfully
    And the reading time should be calculated
```

```python
# tests/bdd/steps/posts.py
from pytest_bdd import scenario, given, when, then

@scenario('../features/posts.feature', 'Create a new post')
def test_create_post():
    pass

@given('I am authenticated as "author@example.com"')
def authenticated_user(api_client, user):
    api_client.force_authenticate(user=user)
    return user

@when('I create a post with title "My First Post"')
def create_post(api_client, context):
    context['response'] = api_client.post('/api/posts/', {
        'title': 'My First Post',
        'content': context['content']
    })
```

### 4. Property Tests (Hypothesis)

**Target:** Edge cases, complex logic validation

```python
# tests/property/test_calculations.py
from hypothesis import given, strategies as st

@given(st.text(min_size=1, max_size=10000))
def test_reading_time_never_zero(content):
    """Property: Reading time is always >= 1"""
    time = calculate_reading_time(content)
    assert time >= 1

@given(st.integers(min_value=1, max_value=1000))
def test_word_count_matches_reading_time(word_count):
    """Property: 200 words = 1 minute"""
    content = " ".join(["word"] * word_count)
    time = calculate_reading_time(content)
    expected = max(1, word_count // 200)
    assert time == expected
```

## Django-Specific Testing

### Database: SQLite (Not PostgreSQL)

```python
# settings/test.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': ':memory:',  # In-memory for speed
    }
}

# ✅ CORRECT - SQLite for tests
# ❌ WRONG - PostgreSQL in tests (slow, complex)
```

### No Mocking Internal Services

```python
# ✅ CORRECT - Use real service
def test_post_creation(db):
    result = post_service.create_post(1, "Title", "Content")
    post = Post.objects.create(**result)
    assert post.reading_time > 0

# ❌ WRONG - Mocking our own code
@patch('services.post_service.create_post')
def test_post_creation(mock_create):  # NO - don't mock internal services
    ...
```

### Real Redis/OpenSearch Test Instances

```python
# ✅ CORRECT - Real test instances
@pytest.fixture(scope='session')
def redis_test():
    """Real Redis on test DB (15)"""
    client = redis.Redis(host='localhost', port=6379, db=15)
    yield client
    client.flushdb()  # Cleanup

# ❌ WRONG - Mocking Redis
@patch('redis.Redis')
def test_cache(mock_redis):  # NO
    ...
```

## Test Execution

```bash
# Fast unit tests only
pytest -m quick

# Integration tests
pytest -m integration

# BDD tests
pytest -m bdd

# All tests with coverage
pytest --cov=services --cov=api --cov-report=html

# Specific test
pytest tests/unit/services/test_post_service.py::TestPostService::test_create_post
```

## Quality Checklist

- [ ] Unit tests for all business logic (services/)
- [ ] Integration tests for all API endpoints
- [ ] BDD tests for user-facing features
- [ ] Property tests for complex calculations
- [ ] Fixtures use SQLite (not PostgreSQL)
- [ ] Real Redis/OpenSearch test instances (not mocked)
- [ ] No mocking of internal services
- [ ] Coverage > 80% on services/ and api/
- [ ] Every bug has a regression test

## Anti-Patterns

❌ Mocking internal services (use SQLite + fixtures)
❌ Mocking Django ORM
❌ Testing framework behavior (DRF serializers, Django forms)
❌ PostgreSQL in tests (use SQLite)
❌ High coverage, low quality (test meaningful paths)
❌ No regression tests for bugs
