# Testing Examples

## Overview

Comprehensive testing examples covering unit, BDD/acceptance, and E2E tests across all technology stacks.

## Testing Matrix

| Stack | Unit Tests (TDD) | BDD/Acceptance | E2E |
|-------|------------------|----------------|-----|
| TALL (Laravel) | Pest PHP | Behat / Pest Stories | Dusk |
| Django | pytest / Django TestCase | Behave / pytest-bdd | Selenium |
| React/Next.js | Jest / Vitest | Cucumber.js / Jest-Cucumber | Cypress / Playwright |
| React Native | Jest | Cucumber.js | Detox |
| Node.js | Jest / Vitest | Cucumber.js | Cypress |

---

## Browser Configuration for E2E Tests (CRITICAL)

**ALWAYS use Chrome for E2E testing. NEVER use Firefox unless explicitly requested.**

### Browser Environment Variable
- **Environment Variable:** `CHROME_PATH` (auto-detected by `chrome-tool.py`)
- **Detection Command:** `./plugins/chrome-tool.py detect`

### Framework-Specific Chrome Configuration

| Framework | Configuration |
|-----------|---------------|
| Laravel Dusk | Uses `DUSK_CHROME_BINARY` from `.env` automatically |
| Playwright | `channel: 'chrome'` or `executablePath: process.env.CHROME_PATH` |
| Cypress | `browser: 'chrome'` in config or `--browser chrome` CLI flag |
| Selenium | `options.binary_location = os.environ.get('CHROME_PATH')` |
| Puppeteer | `executablePath: process.env.PUPPETEER_EXECUTABLE_PATH` |

### Claude Code Chrome Integration

Use `claude --chrome` to enable browser automation for E2E testing:

```bash
# Start Claude Code with Chrome enabled
claude --chrome

# Check connection status
/chrome
```

---

## Table of Contents

- [Testing Examples](#testing-examples)
  - [Overview](#overview)
  - [Testing Matrix](#testing-matrix)
  - [Browser Configuration for E2E Tests (CRITICAL)](#browser-configuration-for-e2e-tests-critical)
    - [Browser Environment Variable](#browser-environment-variable)
    - [Framework-Specific Chrome Configuration](#framework-specific-chrome-configuration)
    - [Claude Code Chrome Integration](#claude-code-chrome-integration)
  - [Table of Contents](#table-of-contents)
  - [TALL Stack (Laravel 12)](#tall-stack-laravel-12)
    - [Unit Tests (Pest)](#unit-tests-pest)
      - [Configuration](#configuration)
      - [Unit Test Examples](#unit-test-examples)
      - [Feature Test Examples](#feature-test-examples)
    - [BDD/Acceptance (Behat)](#bddacceptance-behat)
      - [Configuration](#configuration-1)
      - [Feature Files](#feature-files)
      - [Context Classes](#context-classes)
    - [E2E (Dusk)](#e2e-dusk)
      - [Configuration](#configuration-2)
      - [E2E Test Examples](#e2e-test-examples)
      - [Page Objects](#page-objects)
  - [Django/Wagtail Stack](#djangowagtail-stack)
    - [Unit Tests (Pytest)](#unit-tests-pytest)
      - [Configuration](#configuration-3)
      - [Unit Test Examples](#unit-test-examples-1)
      - [Integration Test Examples](#integration-test-examples)
    - [BDD/Acceptance (Behave)](#bddacceptance-behave)
      - [Configuration](#configuration-4)
      - [Feature Files](#feature-files-1)
      - [Step Definitions](#step-definitions)
    - [E2E (Selenium)](#e2e-selenium)
      - [Configuration](#configuration-5)
      - [E2E Test Examples](#e2e-test-examples-1)
      - [Page Objects](#page-objects-1)
  - [React/Next.js Stack](#reactnextjs-stack)
    - [Unit Tests (Vitest)](#unit-tests-vitest)
      - [Configuration](#configuration-6)
      - [Unit Test Examples](#unit-test-examples-2)
    - [BDD/Acceptance (Cucumber.js)](#bddacceptance-cucumberjs)
      - [Configuration](#configuration-7)
      - [Feature Files](#feature-files-2)
      - [Step Definitions](#step-definitions-1)
    - [E2E (Playwright)](#e2e-playwright)
      - [Configuration](#configuration-8)
      - [E2E Test Examples](#e2e-test-examples-2)
      - [Fixtures and Helpers](#fixtures-and-helpers)
  - [React Native Stack](#react-native-stack)
    - [Unit Tests (Jest)](#unit-tests-jest)
      - [Configuration](#configuration-9)
      - [Unit Test Examples](#unit-test-examples-3)
    - [BDD/Acceptance (Cucumber.js) {#bddacceptance-cucumberjs-rn}](#bddacceptance-cucumberjs-bddacceptance-cucumberjs-rn)
      - [Configuration](#configuration-10)
      - [Feature Files](#feature-files-3)
      - [Step Definitions](#step-definitions-2)
    - [E2E (Detox)](#e2e-detox)
      - [Configuration](#configuration-11)
      - [E2E Test Examples](#e2e-test-examples-3)
  - [Node.js Stack](#nodejs-stack)
    - [Unit Tests (Vitest) {#unit-tests-vitest-node}](#unit-tests-vitest-unit-tests-vitest-node)
      - [Configuration](#configuration-12)
      - [Unit Test Examples](#unit-test-examples-4)
    - [BDD/Acceptance (Cucumber.js) {#bddacceptance-cucumberjs-node}](#bddacceptance-cucumberjs-bddacceptance-cucumberjs-node)
      - [Feature Files](#feature-files-4)
      - [Step Definitions](#step-definitions-3)
    - [E2E (Cypress)](#e2e-cypress)
      - [Configuration](#configuration-13)
      - [E2E Test Examples](#e2e-test-examples-4)
      - [API Testing with Cypress](#api-testing-with-cypress)


## TALL Stack (Laravel 12)

### Unit Tests (Pest)

#### Configuration

```php
<?php
// tests/Pest.php

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\LazilyRefreshDatabase;

uses(LazilyRefreshDatabase::class)->in('Feature');
uses(RefreshDatabase::class)->in('Integration');

expect()->extend('toBeValidEmail', function () {
    return $this->toMatch('/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/');
});

expect()->extend('toBeUuid', function () {
    return $this->toMatch('/^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i');
});

function actingAsUser(?User $user = null): User
{
    $user ??= User::factory()->create();
    test()->actingAs($user);
    return $user;
}

function actingAsAdmin(): User
{
    $user = User::factory()->admin()->create();
    test()->actingAs($user);
    return $user;
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- phpunit.xml -->
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true"
         cacheDirectory=".phpunit.cache"
         executionOrder="depends,defects"
         requireCoverageMetadata="false"
         beStrictAboutOutputDuringTests="true"
         failOnRisky="true"
         failOnWarning="true">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
        <testsuite name="Integration">
            <directory>tests/Integration</directory>
        </testsuite>
    </testsuites>
    <source>
        <include>
            <directory>app</directory>
        </include>
    </source>
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="DB_CONNECTION" value="mysql"/>
        <env name="DB_DATABASE" value="myapp_test"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="QUEUE_CONNECTION" value="sync"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="MAIL_MAILER" value="array"/>
    </php>
</phpunit>
```

#### Unit Test Examples

```php
<?php
// tests/Unit/Services/UserServiceTest.php

use App\Models\User;
use App\Services\UserService;
use App\DTOs\CreateUserDTO;
use App\Exceptions\DuplicateEmailException;

describe('UserService', function () {
    beforeEach(function () {
        $this->service = app(UserService::class);
    });

    describe('createUser', function () {
        it('creates a user with valid data', function () {
            $dto = new CreateUserDTO(
                name: 'John Doe',
                email: 'john@example.com',
                password: 'SecurePass123!'
            );

            $user = $this->service->createUser($dto);

            expect($user)
                ->toBeInstanceOf(User::class)
                ->name->toBe('John Doe')
                ->email->toBe('john@example.com');

            expect(Hash::check('SecurePass123!', $user->password))->toBeTrue();
        });

        it('throws exception for duplicate email', function () {
            User::factory()->create(['email' => 'existing@example.com']);

            $dto = new CreateUserDTO(
                name: 'Jane Doe',
                email: 'existing@example.com',
                password: 'Password123!'
            );

            $this->service->createUser($dto);
        })->throws(DuplicateEmailException::class);

        it('hashes the password securely', function () {
            $dto = new CreateUserDTO(
                name: 'Test User',
                email: 'test@example.com',
                password: 'plaintext'
            );

            $user = $this->service->createUser($dto);

            expect($user->password)
                ->not->toBe('plaintext')
                ->toStartWith('$2y$');
        });
    });
});
```

#### Feature Test Examples

```php
<?php
// tests/Feature/Api/AuthenticationTest.php

use App\Models\User;

describe('Authentication API', function () {
    describe('POST /api/login', function () {
        it('authenticates user with valid credentials', function () {
            $user = User::factory()->create([
                'password' => Hash::make('password123'),
            ]);

            $response = $this->postJson('/api/login', [
                'email' => $user->email,
                'password' => 'password123',
            ]);

            $response->assertOk()
                ->assertJsonStructure([
                    'data' => [
                        'user' => ['id', 'name', 'email'],
                        'token',
                    ],
                ]);
        });

        it('rejects invalid credentials', function () {
            $user = User::factory()->create();

            $response = $this->postJson('/api/login', [
                'email' => $user->email,
                'password' => 'wrongpassword',
            ]);

            $response->assertUnauthorized()
                ->assertJson(['message' => 'Invalid credentials']);
        });

        it('rate limits login attempts', function () {
            $user = User::factory()->create();

            foreach (range(1, 6) as $i) {
                $response = $this->postJson('/api/login', [
                    'email' => $user->email,
                    'password' => 'wrongpassword',
                ]);
            }

            $response->assertTooManyRequests();
        });
    });
});
```

### BDD/Acceptance (Behat)

#### Configuration

```yaml
# behat.yml

default:
    suites:
        default:
            contexts:
                - FeatureContext
                - Behat\MinkExtension\Context\MinkContext
                - App\Tests\Behat\ApiContext
                - App\Tests\Behat\DatabaseContext
            filters:
                tags: "~@wip"

    extensions:
        Behat\MinkExtension:
            base_url: 'http://localhost:8000'
            default_session: laravel
            laravel: ~
            sessions:
                laravel:
                    laravel: ~
                javascript:
                    selenium2:
                        wd_host: 'http://localhost:4444/wd/hub'
                        browser: chrome

        Laracasts\Behat\ServiceContainer\BehatExtension:
            env_path: .env.testing
```

#### Feature Files

```gherkin
# features/authentication/login.feature

Feature: User Authentication
    As a registered user
    I want to log in to my account
    So that I can access protected features

    Background:
        Given I am on the login page

    @smoke
    Scenario: Successful login with valid credentials
        Given a user exists with email "user@example.com" and password "SecurePass123!"
        When I fill in "email" with "user@example.com"
        And I fill in "password" with "SecurePass123!"
        And I press "Log in"
        Then I should be on the dashboard page
        And I should see "Welcome back"

    Scenario: Failed login with invalid password
        Given a user exists with email "user@example.com" and password "SecurePass123!"
        When I fill in "email" with "user@example.com"
        And I fill in "password" with "wrongpassword"
        And I press "Log in"
        Then I should see "Invalid credentials"
        And I should be on the login page

    Scenario: Login form validation
        When I press "Log in"
        Then I should see "The email field is required"
        And I should see "The password field is required"

    @security
    Scenario: Rate limiting after failed attempts
        Given a user exists with email "user@example.com" and password "SecurePass123!"
        When I attempt to login 6 times with wrong password
        Then I should see "Too many login attempts"
```

```gherkin
# features/authentication/registration.feature

Feature: User Registration
    As a visitor
    I want to create an account
    So that I can access the application

    @smoke
    Scenario: Successful registration
        Given I am on the registration page
        When I fill in the following:
            | name     | John Doe              |
            | email    | john@example.com      |
            | password | SecurePassword123!    |
            | password_confirmation | SecurePassword123! |
        And I press "Register"
        Then I should be on the dashboard page
        And I should see "Welcome, John Doe"
        And a confirmation email should be sent to "john@example.com"

    Scenario Outline: Registration validation
        Given I am on the registration page
        When I fill in "email" with "<email>"
        And I fill in "password" with "<password>"
        And I press "Register"
        Then I should see "<error_message>"

        Examples:
            | email            | password      | error_message                    |
            |                  | SecurePass1!  | The email field is required      |
            | invalid-email    | SecurePass1!  | The email must be a valid email  |
            | user@example.com |               | The password field is required   |
            | user@example.com | short         | The password must be at least 8  |
```

#### Context Classes

```php
<?php
// tests/Behat/ApiContext.php

namespace App\Tests\Behat;

use Behat\Behat\Context\Context;
use Illuminate\Support\Facades\Http;
use PHPUnit\Framework\Assert;

class ApiContext implements Context
{
    private array $response = [];
    private int $statusCode = 0;
    private string $token = '';

    /**
     * @Given I am authenticated as :email
     */
    public function iAmAuthenticatedAs(string $email): void
    {
        $response = Http::post('/api/login', [
            'email' => $email,
            'password' => 'password123',
        ]);

        $this->token = $response->json('data.token');
    }

    /**
     * @When I send a :method request to :endpoint
     */
    public function iSendARequestTo(string $method, string $endpoint): void
    {
        $headers = [];
        if ($this->token) {
            $headers['Authorization'] = "Bearer {$this->token}";
        }

        $response = Http::withHeaders($headers)->{strtolower($method)}($endpoint);
        $this->statusCode = $response->status();
        $this->response = $response->json() ?? [];
    }

    /**
     * @When I send a :method request to :endpoint with body:
     */
    public function iSendARequestToWithBody(string $method, string $endpoint, \Behat\Gherkin\Node\PyStringNode $body): void
    {
        $headers = ['Content-Type' => 'application/json'];
        if ($this->token) {
            $headers['Authorization'] = "Bearer {$this->token}";
        }

        $response = Http::withHeaders($headers)
            ->{strtolower($method)}($endpoint, json_decode($body->getRaw(), true));

        $this->statusCode = $response->status();
        $this->response = $response->json() ?? [];
    }

    /**
     * @Then the response status code should be :code
     */
    public function theResponseStatusCodeShouldBe(int $code): void
    {
        Assert::assertEquals($code, $this->statusCode);
    }

    /**
     * @Then the response should contain :key with value :value
     */
    public function theResponseShouldContainWithValue(string $key, string $value): void
    {
        Assert::assertEquals($value, data_get($this->response, $key));
    }

    /**
     * @Then the response should have a :key field
     */
    public function theResponseShouldHaveAField(string $key): void
    {
        Assert::assertTrue(
            data_get($this->response, $key) !== null,
            "Expected response to have '{$key}' field"
        );
    }
}
```

```php
<?php
// tests/Behat/DatabaseContext.php

namespace App\Tests\Behat;

use Behat\Behat\Context\Context;
use App\Models\User;
use Illuminate\Support\Facades\Hash;
use PHPUnit\Framework\Assert;

class DatabaseContext implements Context
{
    /**
     * @Given a user exists with email :email and password :password
     */
    public function aUserExistsWithEmailAndPassword(string $email, string $password): void
    {
        User::factory()->create([
            'email' => $email,
            'password' => Hash::make($password),
        ]);
    }

    /**
     * @Given the following users exist:
     */
    public function theFollowingUsersExist(\Behat\Gherkin\Node\TableNode $table): void
    {
        foreach ($table->getHash() as $row) {
            User::factory()->create([
                'name' => $row['name'],
                'email' => $row['email'],
                'password' => Hash::make($row['password'] ?? 'password123'),
            ]);
        }
    }

    /**
     * @Then a user should exist with email :email
     */
    public function aUserShouldExistWithEmail(string $email): void
    {
        Assert::assertTrue(
            User::where('email', $email)->exists(),
            "Expected user with email '{$email}' to exist"
        );
    }

    /**
     * @Then a confirmation email should be sent to :email
     */
    public function aConfirmationEmailShouldBeSentTo(string $email): void
    {
        // Check mail was sent (using Laravel's mail fake)
        \Illuminate\Support\Facades\Mail::assertSent(
            \App\Mail\WelcomeEmail::class,
            fn ($mail) => $mail->hasTo($email)
        );
    }
}
```

### E2E (Dusk)

#### Configuration

**IMPORTANT:** Use Chrome or Chrome Beta. Set `DUSK_CHROME_BINARY` environment variable to specify Chrome Beta.

```php
<?php
// tests/DuskTestCase.php

namespace Tests;

use Laravel\Dusk\TestCase as BaseTestCase;
use Facebook\WebDriver\Chrome\ChromeOptions;
use Facebook\WebDriver\Remote\RemoteWebDriver;
use Facebook\WebDriver\Remote\DesiredCapabilities;

abstract class DuskTestCase extends BaseTestCase
{
    use CreatesApplication;

    protected static ?string $baseUrl = 'http://localhost:8000';

    public static function prepare(): void
    {
        if (!static::runningInSail()) {
            static::startChromeDriver(['--port=9515']);
        }
    }

    protected function driver(): RemoteWebDriver
    {
        $options = (new ChromeOptions)->addArguments([
            '--disable-gpu',
            '--headless=new',
            '--window-size=1920,1080',
            '--no-sandbox',
            '--disable-dev-shm-usage',
        ]);

        // Use Chrome Beta if DUSK_CHROME_BINARY is set
        if ($binary = env('DUSK_CHROME_BINARY')) {
            $options->setBinary($binary);
        }

        return RemoteWebDriver::create(
            'http://localhost:9515',
            DesiredCapabilities::chrome()->setCapability(
                ChromeOptions::CAPABILITY, $options
            )
        );
    }
}
```

```bash
# Chrome path is auto-detected and set in .env.chrome
# Run: ./plugins/chrome-tool.py write

# Or add to .env.testing (uses environment variable)
DUSK_CHROME_BINARY=${CHROME_PATH}
```

#### E2E Test Examples

```php
<?php
// tests/Browser/AuthenticationTest.php

use App\Models\User;
use Laravel\Dusk\Browser;

describe('Authentication E2E', function () {
    describe('Login Flow', function () {
        it('allows user to log in successfully', function () {
            $user = User::factory()->create([
                'password' => Hash::make('password'),
            ]);

            $this->browse(function (Browser $browser) use ($user) {
                $browser->visit('/login')
                    ->type('email', $user->email)
                    ->type('password', 'password')
                    ->press('Log in')
                    ->assertPathIs('/dashboard')
                    ->assertSee('Welcome back');
            });
        });

        it('shows validation errors for empty fields', function () {
            $this->browse(function (Browser $browser) {
                $browser->visit('/login')
                    ->press('Log in')
                    ->assertSee('The email field is required')
                    ->assertSee('The password field is required');
            });
        });

        it('handles invalid credentials gracefully', function () {
            $user = User::factory()->create();

            $this->browse(function (Browser $browser) use ($user) {
                $browser->visit('/login')
                    ->type('email', $user->email)
                    ->type('password', 'wrongpassword')
                    ->press('Log in')
                    ->assertPathIs('/login')
                    ->assertSee('Invalid credentials');
            });
        });
    });

    describe('Registration Flow', function () {
        it('allows new user to register', function () {
            $this->browse(function (Browser $browser) {
                $browser->visit('/register')
                    ->type('name', 'New User')
                    ->type('email', 'newuser@example.com')
                    ->type('password', 'SecurePass123!')
                    ->type('password_confirmation', 'SecurePass123!')
                    ->press('Register')
                    ->assertPathIs('/dashboard')
                    ->assertSee('Welcome, New User');
            });

            $this->assertDatabaseHas('users', [
                'email' => 'newuser@example.com',
            ]);
        });
    });

    describe('Password Reset Flow', function () {
        it('sends password reset email', function () {
            $user = User::factory()->create();

            $this->browse(function (Browser $browser) use ($user) {
                $browser->visit('/forgot-password')
                    ->type('email', $user->email)
                    ->press('Send Reset Link')
                    ->assertSee('Password reset link sent');
            });
        });
    });
});
```

#### Page Objects

```php
<?php
// tests/Browser/Pages/LoginPage.php

namespace Tests\Browser\Pages;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Page;

class LoginPage extends Page
{
    public function url(): string
    {
        return '/login';
    }

    public function assert(Browser $browser): void
    {
        $browser->assertPathIs($this->url())
            ->assertSee('Log in');
    }

    public function elements(): array
    {
        return [
            '@email' => 'input[name="email"]',
            '@password' => 'input[name="password"]',
            '@submit' => 'button[type="submit"]',
            '@forgot-password' => 'a[href*="forgot-password"]',
        ];
    }

    public function loginAs(Browser $browser, string $email, string $password): void
    {
        $browser->type('@email', $email)
            ->type('@password', $password)
            ->click('@submit');
    }
}
```

---

## Django/Wagtail Stack

### Unit Tests (Pytest)

#### Configuration

```toml
# pyproject.toml

[tool.pytest.ini_options]
DJANGO_SETTINGS_MODULE = "config.settings.testing"
python_files = ["test_*.py", "*_test.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = [
    "-v",
    "--tb=short",
    "--strict-markers",
    "--reuse-db",
    "-p", "no:warnings",
]
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
    "integration: marks integration tests",
    "e2e: marks end-to-end tests",
]
filterwarnings = [
    "ignore::DeprecationWarning",
]
```

```python
# conftest.py

import pytest
from django.contrib.auth import get_user_model
from rest_framework.test import APIClient

User = get_user_model()


@pytest.fixture
def api_client() -> APIClient:
    """Return an unauthenticated API client."""
    return APIClient()


@pytest.fixture
def user(db) -> User:
    """Create and return a regular user."""
    return User.objects.create_user(
        email='user@example.com',
        password='testpass123',
        first_name='Test',
        last_name='User',
    )


@pytest.fixture
def admin_user(db) -> User:
    """Create and return an admin user."""
    return User.objects.create_superuser(
        email='admin@example.com',
        password='adminpass123',
        first_name='Admin',
        last_name='User',
    )


@pytest.fixture
def authenticated_client(api_client: APIClient, user: User) -> APIClient:
    """Return an authenticated API client."""
    api_client.force_authenticate(user=user)
    return api_client
```

#### Unit Test Examples

```python
# apps/users/tests/test_services.py

import pytest
from unittest.mock import Mock, patch
from apps.users.services import UserService
from apps.users.models import User
from apps.users.exceptions import DuplicateEmailError


class TestUserService:
    """Tests for UserService."""

    @pytest.fixture
    def service(self) -> UserService:
        return UserService()

    class TestCreateUser:
        """Tests for create_user method."""

        def test_creates_user_with_valid_data(self, service: UserService, db):
            """Should create user with valid data."""
            user = service.create_user(
                email='new@example.com',
                password='SecurePass123!',
                first_name='John',
                last_name='Doe',
            )

            assert user.pk is not None
            assert user.email == 'new@example.com'
            assert user.first_name == 'John'
            assert user.check_password('SecurePass123!')

        def test_raises_error_for_duplicate_email(
            self, service: UserService, user: User
        ):
            """Should raise error when email already exists."""
            with pytest.raises(DuplicateEmailError):
                service.create_user(
                    email=user.email,
                    password='Password123!',
                    first_name='Jane',
                    last_name='Doe',
                )

        def test_normalises_email(self, service: UserService, db):
            """Should normalise email address."""
            user = service.create_user(
                email='Test@EXAMPLE.COM',
                password='Password123!',
                first_name='Test',
                last_name='User',
            )

            assert user.email == 'Test@example.com'
```

#### Integration Test Examples

```python
# apps/users/tests/test_api.py

import pytest
from django.urls import reverse
from rest_framework import status


class TestAuthenticationAPI:
    """Integration tests for authentication endpoints."""

    class TestLogin:
        """Tests for login endpoint."""

        @pytest.fixture
        def url(self) -> str:
            return reverse('api:login')

        def test_authenticates_with_valid_credentials(
            self, api_client, user, url
        ):
            """Should return token for valid credentials."""
            response = api_client.post(url, {
                'email': user.email,
                'password': 'testpass123',
            })

            assert response.status_code == status.HTTP_200_OK
            assert 'token' in response.data
            assert response.data['user']['email'] == user.email

        def test_rejects_invalid_credentials(self, api_client, user, url):
            """Should return 401 for invalid credentials."""
            response = api_client.post(url, {
                'email': user.email,
                'password': 'wrongpassword',
            })

            assert response.status_code == status.HTTP_401_UNAUTHORIZED
```

### BDD/Acceptance (Behave)

#### Configuration

```ini
# behave.ini

[behave]
paths = features
format = progress
show_skipped = false
show_source = true
show_timings = true
stdout_capture = false
stderr_capture = false
log_capture = false
```

```python
# features/environment.py

from django.test.utils import setup_test_environment, teardown_test_environment
from django.core.management import call_command
from django.test import Client
import django


def before_all(context):
    """Set up Django test environment."""
    django.setup()
    setup_test_environment()
    context.client = Client()


def before_scenario(context, scenario):
    """Reset database before each scenario."""
    call_command('flush', '--no-input', verbosity=0)


def after_all(context):
    """Tear down Django test environment."""
    teardown_test_environment()
```

#### Feature Files

```gherkin
# features/authentication/login.feature

Feature: User Login
    As a registered user
    I want to log in to my account
    So that I can access protected features

    Background:
        Given the following users exist:
            | email              | password       | first_name |
            | user@example.com   | SecurePass123! | John       |

    @smoke
    Scenario: Successful login
        Given I am on the login page
        When I enter "user@example.com" as email
        And I enter "SecurePass123!" as password
        And I click the login button
        Then I should be redirected to the dashboard
        And I should see "Welcome back, John"

    Scenario: Failed login with wrong password
        Given I am on the login page
        When I enter "user@example.com" as email
        And I enter "wrongpassword" as password
        And I click the login button
        Then I should see an error message "Invalid credentials"
        And I should remain on the login page

    Scenario: Login form validation
        Given I am on the login page
        When I click the login button without entering credentials
        Then I should see validation error for "email"
        And I should see validation error for "password"
```

```gherkin
# features/api/users.feature

Feature: User API
    As an API consumer
    I want to manage users via the API
    So that I can integrate with external systems

    Background:
        Given I am authenticated as an admin user

    @api
    Scenario: List all users
        Given the following users exist:
            | email           | first_name |
            | alice@test.com  | Alice      |
            | bob@test.com    | Bob        |
        When I send a GET request to "/api/users/"
        Then the response status should be 200
        And the response should contain 2 users

    @api
    Scenario: Create a new user
        When I send a POST request to "/api/users/" with:
            """
            {
                "email": "newuser@example.com",
                "first_name": "New",
                "last_name": "User",
                "password": "SecurePass123!"
            }
            """
        Then the response status should be 201
        And a user should exist with email "newuser@example.com"
```

#### Step Definitions

```python
# features/steps/authentication_steps.py

from behave import given, when, then
from django.urls import reverse
from django.contrib.auth import get_user_model

User = get_user_model()


@given('the following users exist')
def step_create_users(context):
    """Create users from table."""
    for row in context.table:
        User.objects.create_user(
            email=row['email'],
            password=row['password'],
            first_name=row.get('first_name', 'Test'),
            last_name=row.get('last_name', 'User'),
        )


@given('I am on the login page')
def step_visit_login_page(context):
    """Navigate to login page."""
    context.response = context.client.get(reverse('login'))
    assert context.response.status_code == 200


@when('I enter "{value}" as email')
def step_enter_email(context, value):
    """Store email for login."""
    if not hasattr(context, 'login_data'):
        context.login_data = {}
    context.login_data['email'] = value


@when('I enter "{value}" as password')
def step_enter_password(context, value):
    """Store password for login."""
    if not hasattr(context, 'login_data'):
        context.login_data = {}
    context.login_data['password'] = value


@when('I click the login button')
def step_click_login(context):
    """Submit login form."""
    context.response = context.client.post(
        reverse('login'),
        context.login_data,
        follow=True,
    )


@then('I should be redirected to the dashboard')
def step_verify_dashboard_redirect(context):
    """Verify redirect to dashboard."""
    assert context.response.redirect_chain[-1][0] == reverse('dashboard')


@then('I should see "{text}"')
def step_verify_text_visible(context, text):
    """Verify text is visible on page."""
    assert text.encode() in context.response.content


@then('I should see an error message "{message}"')
def step_verify_error_message(context, message):
    """Verify error message is displayed."""
    assert message.encode() in context.response.content
```

```python
# features/steps/api_steps.py

from behave import given, when, then
import json
from django.contrib.auth import get_user_model
from rest_framework.test import APIClient

User = get_user_model()


@given('I am authenticated as an admin user')
def step_authenticate_admin(context):
    """Create and authenticate as admin."""
    admin = User.objects.create_superuser(
        email='admin@example.com',
        password='adminpass123',
    )
    context.api_client = APIClient()
    context.api_client.force_authenticate(user=admin)


@when('I send a {method} request to "{endpoint}"')
def step_send_request(context, method, endpoint):
    """Send API request."""
    method_func = getattr(context.api_client, method.lower())
    context.response = method_func(endpoint)


@when('I send a {method} request to "{endpoint}" with')
def step_send_request_with_body(context, method, endpoint):
    """Send API request with JSON body."""
    method_func = getattr(context.api_client, method.lower())
    data = json.loads(context.text)
    context.response = method_func(endpoint, data, format='json')


@then('the response status should be {status:d}')
def step_verify_status(context, status):
    """Verify response status code."""
    assert context.response.status_code == status


@then('the response should contain {count:d} users')
def step_verify_user_count(context, count):
    """Verify number of users in response."""
    assert len(context.response.data) == count


@then('a user should exist with email "{email}"')
def step_verify_user_exists(context, email):
    """Verify user exists in database."""
    assert User.objects.filter(email=email).exists()
```

### E2E (Selenium)

#### Configuration

**IMPORTANT:** Use Chrome or Chrome Beta for Selenium tests.

```python
# tests/e2e/conftest.py

import os
import pytest
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
from django.contrib.staticfiles.testing import StaticLiveServerTestCase

# Chrome binary path (auto-detected via chrome-tool.py)
CHROME_BINARY = os.environ.get('CHROME_PATH')


@pytest.fixture(scope='session')
def chrome_options():
    """Chrome options for headless testing."""
    options = Options()
    options.binary_location = CHROME_BINARY  # Use Chrome Beta
    options.add_argument('--headless=new')
    options.add_argument('--no-sandbox')
    options.add_argument('--disable-dev-shm-usage')
    options.add_argument('--window-size=1920,1080')
    return options


@pytest.fixture
def browser(chrome_options):
    """Create and return a Chrome WebDriver instance."""
    service = Service(ChromeDriverManager().install())
    driver = webdriver.Chrome(service=service, options=chrome_options)
    driver.implicitly_wait(10)
    yield driver
    driver.quit()


@pytest.fixture
def live_server(db):
    """Create a live server for E2E tests."""
    server = StaticLiveServerTestCase()
    server._pre_setup()
    server._setup_live_server()
    yield server.live_server_url
    server._teardown_live_server()
    server._post_teardown()
```

```bash
# Chrome path is auto-detected via chrome-tool.py
# Run: ./plugins/chrome-tool.py write
# Then source the generated .env.chrome file
```

#### E2E Test Examples

```python
# tests/e2e/test_authentication.py

import pytest
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from django.contrib.auth import get_user_model

User = get_user_model()


class TestLoginE2E:
    """E2E tests for login functionality."""

    @pytest.fixture(autouse=True)
    def setup(self, db):
        """Create test user."""
        self.user = User.objects.create_user(
            email='e2e@example.com',
            password='TestPass123!',
            first_name='E2E',
            last_name='User',
        )

    def test_successful_login(self, browser, live_server):
        """User can log in with valid credentials."""
        browser.get(f'{live_server}/login/')

        # Fill in login form
        browser.find_element(By.NAME, 'email').send_keys('e2e@example.com')
        browser.find_element(By.NAME, 'password').send_keys('TestPass123!')
        browser.find_element(By.CSS_SELECTOR, 'button[type="submit"]').click()

        # Wait for redirect to dashboard
        WebDriverWait(browser, 10).until(
            EC.url_contains('/dashboard/')
        )

        # Verify welcome message
        welcome = browser.find_element(By.CLASS_NAME, 'welcome-message')
        assert 'Welcome back, E2E' in welcome.text

    def test_login_with_invalid_credentials(self, browser, live_server):
        """Shows error for invalid credentials."""
        browser.get(f'{live_server}/login/')

        browser.find_element(By.NAME, 'email').send_keys('e2e@example.com')
        browser.find_element(By.NAME, 'password').send_keys('wrongpassword')
        browser.find_element(By.CSS_SELECTOR, 'button[type="submit"]').click()

        # Wait for error message
        error = WebDriverWait(browser, 10).until(
            EC.visibility_of_element_located((By.CLASS_NAME, 'error-message'))
        )

        assert 'Invalid credentials' in error.text

    def test_login_form_validation(self, browser, live_server):
        """Shows validation errors for empty fields."""
        browser.get(f'{live_server}/login/')

        # Submit empty form
        browser.find_element(By.CSS_SELECTOR, 'button[type="submit"]').click()

        # Wait for validation errors
        WebDriverWait(browser, 10).until(
            EC.visibility_of_element_located((By.CLASS_NAME, 'field-error'))
        )

        errors = browser.find_elements(By.CLASS_NAME, 'field-error')
        error_texts = [e.text for e in errors]

        assert any('email' in t.lower() for t in error_texts)
        assert any('password' in t.lower() for t in error_texts)


class TestRegistrationE2E:
    """E2E tests for registration functionality."""

    def test_successful_registration(self, browser, live_server, db):
        """New user can register successfully."""
        browser.get(f'{live_server}/register/')

        # Fill in registration form
        browser.find_element(By.NAME, 'first_name').send_keys('New')
        browser.find_element(By.NAME, 'last_name').send_keys('User')
        browser.find_element(By.NAME, 'email').send_keys('newuser@example.com')
        browser.find_element(By.NAME, 'password1').send_keys('SecurePass123!')
        browser.find_element(By.NAME, 'password2').send_keys('SecurePass123!')
        browser.find_element(By.CSS_SELECTOR, 'button[type="submit"]').click()

        # Wait for redirect to dashboard
        WebDriverWait(browser, 10).until(
            EC.url_contains('/dashboard/')
        )

        # Verify user was created
        assert User.objects.filter(email='newuser@example.com').exists()
```

#### Page Objects

```python
# tests/e2e/pages/login_page.py

from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC


class LoginPage:
    """Page object for login page."""

    URL = '/login/'

    # Locators
    EMAIL_INPUT = (By.NAME, 'email')
    PASSWORD_INPUT = (By.NAME, 'password')
    SUBMIT_BUTTON = (By.CSS_SELECTOR, 'button[type="submit"]')
    ERROR_MESSAGE = (By.CLASS_NAME, 'error-message')
    FIELD_ERROR = (By.CLASS_NAME, 'field-error')

    def __init__(self, browser, base_url):
        self.browser = browser
        self.base_url = base_url
        self.wait = WebDriverWait(browser, 10)

    def navigate(self):
        """Navigate to login page."""
        self.browser.get(f'{self.base_url}{self.URL}')
        return self

    def enter_email(self, email: str):
        """Enter email address."""
        self.browser.find_element(*self.EMAIL_INPUT).send_keys(email)
        return self

    def enter_password(self, password: str):
        """Enter password."""
        self.browser.find_element(*self.PASSWORD_INPUT).send_keys(password)
        return self

    def submit(self):
        """Click submit button."""
        self.browser.find_element(*self.SUBMIT_BUTTON).click()
        return self

    def login(self, email: str, password: str):
        """Complete login flow."""
        return self.enter_email(email).enter_password(password).submit()

    def get_error_message(self) -> str:
        """Get error message text."""
        error = self.wait.until(
            EC.visibility_of_element_located(self.ERROR_MESSAGE)
        )
        return error.text

    def get_field_errors(self) -> list[str]:
        """Get all field error messages."""
        self.wait.until(
            EC.visibility_of_element_located(self.FIELD_ERROR)
        )
        errors = self.browser.find_elements(*self.FIELD_ERROR)
        return [e.text for e in errors]
```

---

## React/Next.js Stack

### Unit Tests (Vitest)

#### Configuration

```typescript
// vitest.config.ts

import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./tests/setup.ts'],
    include: ['**/*.{test,spec}.{ts,tsx}'],
    exclude: ['node_modules', '.next', 'e2e'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules',
        'tests',
        '**/*.d.ts',
        '**/*.config.*',
      ],
    },
    mockReset: true,
    restoreMocks: true,
  },
});
```

```typescript
// tests/setup.ts

import '@testing-library/jest-dom/vitest';
import { cleanup } from '@testing-library/react';
import { afterEach, vi } from 'vitest';

afterEach(() => {
  cleanup();
});

// Mock next/navigation
vi.mock('next/navigation', () => ({
  useRouter: () => ({
    push: vi.fn(),
    replace: vi.fn(),
    prefetch: vi.fn(),
    back: vi.fn(),
  }),
  useSearchParams: () => new URLSearchParams(),
  usePathname: () => '/',
}));

// Mock next/image
vi.mock('next/image', () => ({
  default: ({ src, alt, ...props }: { src: string; alt: string }) => (
    <img src={src} alt={alt} {...props} />
  ),
}));
```

#### Unit Test Examples

```typescript
// components/LoginForm/LoginForm.test.tsx

import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  const mockOnSubmit = vi.fn();

  beforeEach(() => {
    mockOnSubmit.mockClear();
  });

  it('renders email and password fields', () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);

    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /log in/i })).toBeInTheDocument();
  });

  it('calls onSubmit with form data', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={mockOnSubmit} />);

    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /log in/i }));

    await waitFor(() => {
      expect(mockOnSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123',
      });
    });
  });

  it('displays validation errors', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={mockOnSubmit} />);

    await user.click(screen.getByRole('button', { name: /log in/i }));

    await waitFor(() => {
      expect(screen.getByText(/email is required/i)).toBeInTheDocument();
      expect(screen.getByText(/password is required/i)).toBeInTheDocument();
    });

    expect(mockOnSubmit).not.toHaveBeenCalled();
  });

  it('disables submit button while loading', () => {
    render(<LoginForm onSubmit={mockOnSubmit} isLoading />);

    expect(screen.getByRole('button', { name: /logging in/i })).toBeDisabled();
  });
});
```

### BDD/Acceptance (Cucumber.js)

#### Configuration

```javascript
// cucumber.js

module.exports = {
  default: {
    require: ['features/step_definitions/**/*.ts', 'features/support/**/*.ts'],
    requireModule: ['ts-node/register'],
    format: ['progress-bar', 'html:reports/cucumber.html'],
    formatOptions: { snippetInterface: 'async-await' },
    publishQuiet: true,
  },
};
```

```typescript
// features/support/world.ts

import { setWorldConstructor, World, IWorldOptions } from '@cucumber/cucumber';
import { Browser, chromium, Page } from '@playwright/test';

export interface CustomWorld extends World {
  browser: Browser;
  page: Page;
  baseUrl: string;
}

class CustomWorldImpl extends World implements CustomWorld {
  browser!: Browser;
  page!: Page;
  baseUrl = process.env.BASE_URL || 'http://localhost:3000';

  constructor(options: IWorldOptions) {
    super(options);
  }

  async init(): Promise<void> {
    this.browser = await chromium.launch({ headless: true });
    const context = await this.browser.newContext();
    this.page = await context.newPage();
  }

  async cleanup(): Promise<void> {
    await this.browser.close();
  }
}

setWorldConstructor(CustomWorldImpl);
```

```typescript
// features/support/hooks.ts

import { Before, After, BeforeAll, AfterAll } from '@cucumber/cucumber';
import { CustomWorld } from './world';

Before(async function (this: CustomWorld) {
  await this.init();
});

After(async function (this: CustomWorld) {
  await this.cleanup();
});
```

#### Feature Files

```gherkin
# features/authentication/login.feature

Feature: User Login
    As a registered user
    I want to log in to my account
    So that I can access protected features

    Background:
        Given I am on the login page

    @smoke
    Scenario: Successful login
        When I enter "user@example.com" as email
        And I enter "SecurePass123!" as password
        And I click the login button
        Then I should be redirected to the dashboard
        And I should see "Welcome back"

    Scenario: Failed login with invalid credentials
        When I enter "user@example.com" as email
        And I enter "wrongpassword" as password
        And I click the login button
        Then I should see an error "Invalid credentials"

    Scenario Outline: Form validation
        When I enter "<email>" as email
        And I enter "<password>" as password
        And I click the login button
        Then I should see validation error "<error>"

        Examples:
            | email            | password | error                      |
            |                  | pass123  | Email is required          |
            | invalid          | pass123  | Please enter a valid email |
            | test@example.com |          | Password is required       |
```

#### Step Definitions

```typescript
// features/step_definitions/authentication.steps.ts

import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { CustomWorld } from '../support/world';

Given('I am on the login page', async function (this: CustomWorld) {
  await this.page.goto(`${this.baseUrl}/login`);
});

When('I enter {string} as email', async function (this: CustomWorld, email: string) {
  await this.page.getByLabel('Email').fill(email);
});

When('I enter {string} as password', async function (this: CustomWorld, password: string) {
  await this.page.getByLabel('Password').fill(password);
});

When('I click the login button', async function (this: CustomWorld) {
  await this.page.getByRole('button', { name: 'Log in' }).click();
});

Then('I should be redirected to the dashboard', async function (this: CustomWorld) {
  await expect(this.page).toHaveURL(/\/dashboard/);
});

Then('I should see {string}', async function (this: CustomWorld, text: string) {
  await expect(this.page.getByText(text)).toBeVisible();
});

Then('I should see an error {string}', async function (this: CustomWorld, error: string) {
  await expect(this.page.getByRole('alert')).toContainText(error);
});

Then('I should see validation error {string}', async function (this: CustomWorld, error: string) {
  await expect(this.page.getByText(error)).toBeVisible();
});
```

### E2E (Playwright)

#### Configuration

**IMPORTANT:** Use Chrome for Playwright tests. Set `channel: 'chrome'` to use the installed Chrome browser.

```typescript
// playwright.config.ts

import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html', { open: 'never' }],
    ['json', { outputFile: 'reports/results.json' }],
  ],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    // Use installed Chrome browser (CRITICAL)
    channel: 'chrome',
  },
  projects: [
    {
      name: 'chrome',
      use: {
        ...devices['Desktop Chrome'],
        channel: 'chrome',  // Use installed Chrome
      },
    },
    {
      name: 'chrome-custom',
      use: {
        ...devices['Desktop Chrome'],
        launchOptions: {
          executablePath: process.env.CHROME_PATH,
        },
      },
    },
    {
      name: 'mobile-chrome',
      use: {
        ...devices['Pixel 5'],
        channel: 'chrome',
      },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

```bash
# Run Playwright with Chrome
npx playwright test --project=chrome

# Run Playwright with Chrome Beta
npx playwright test --project=chrome-beta
```

#### E2E Test Examples

```typescript
// e2e/authentication.spec.ts

import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test.describe('Login', () => {
    test('allows user to log in with valid credentials', async ({ page }) => {
      await page.goto('/login');

      await page.getByLabel('Email').fill('test@example.com');
      await page.getByLabel('Password').fill('password123');
      await page.getByRole('button', { name: 'Log in' }).click();

      await expect(page).toHaveURL('/dashboard');
      await expect(page.getByText('Welcome back')).toBeVisible();
    });

    test('shows error for invalid credentials', async ({ page }) => {
      await page.goto('/login');

      await page.getByLabel('Email').fill('test@example.com');
      await page.getByLabel('Password').fill('wrongpassword');
      await page.getByRole('button', { name: 'Log in' }).click();

      await expect(page.getByRole('alert')).toContainText('Invalid credentials');
      await expect(page).toHaveURL('/login');
    });

    test('validates required fields', async ({ page }) => {
      await page.goto('/login');
      await page.getByRole('button', { name: 'Log in' }).click();

      await expect(page.getByText('Email is required')).toBeVisible();
      await expect(page.getByText('Password is required')).toBeVisible();
    });

    test('redirects authenticated users to dashboard', async ({ page, context }) => {
      // Set auth cookie
      await context.addCookies([
        {
          name: 'auth-token',
          value: 'valid-token',
          domain: 'localhost',
          path: '/',
        },
      ]);

      await page.goto('/login');

      await expect(page).toHaveURL('/dashboard');
    });
  });

  test.describe('Logout', () => {
    test.beforeEach(async ({ page }) => {
      // Login first
      await page.goto('/login');
      await page.getByLabel('Email').fill('test@example.com');
      await page.getByLabel('Password').fill('password123');
      await page.getByRole('button', { name: 'Log in' }).click();
      await expect(page).toHaveURL('/dashboard');
    });

    test('logs out user successfully', async ({ page }) => {
      await page.getByRole('button', { name: 'Logout' }).click();

      await expect(page).toHaveURL('/');
      await expect(page.getByRole('link', { name: 'Log in' })).toBeVisible();
    });
  });
});
```

#### Fixtures and Helpers

```typescript
// e2e/fixtures/auth.fixture.ts

import { test as base, Page } from '@playwright/test';

interface AuthFixtures {
  authenticatedPage: Page;
}

export const test = base.extend<AuthFixtures>({
  authenticatedPage: async ({ page, context }, use) => {
    // Set up authentication
    await context.addCookies([
      {
        name: 'auth-token',
        value: 'test-token',
        domain: 'localhost',
        path: '/',
      },
    ]);

    await use(page);
  },
});

export { expect } from '@playwright/test';
```

---

## React Native Stack

### Unit Tests (Jest)

#### Configuration

```javascript
// jest.config.js

module.exports = {
  preset: 'jest-expo',
  setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@sentry/react-native|native-base|react-native-svg)',
  ],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/types.ts',
  ],
  testMatch: ['**/__tests__/**/*.[jt]s?(x)', '**/?(*.)+(spec|test).[jt]s?(x)'],
  testPathIgnorePatterns: ['/node_modules/', '/e2e/'],
};
```

```typescript
// tests/setup.ts

import '@testing-library/react-native/extend-expect';
import { jest } from '@jest/globals';

// Mock expo modules
jest.mock('expo-secure-store', () => ({
  getItemAsync: jest.fn(),
  setItemAsync: jest.fn(),
  deleteItemAsync: jest.fn(),
}));

jest.mock('expo-router', () => ({
  useRouter: () => ({
    push: jest.fn(),
    replace: jest.fn(),
    back: jest.fn(),
  }),
  useLocalSearchParams: () => ({}),
  useSegments: () => [],
}));

// Silence the warning: Animated: `useNativeDriver`
jest.mock('react-native/Libraries/Animated/NativeAnimatedHelper');
```

#### Unit Test Examples

```typescript
// src/components/LoginForm/__tests__/LoginForm.test.tsx

import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react-native';
import { LoginForm } from '../LoginForm';

describe('LoginForm', () => {
  const mockOnSubmit = jest.fn();

  beforeEach(() => {
    mockOnSubmit.mockClear();
  });

  it('renders email and password inputs', () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);

    expect(screen.getByPlaceholderText('Email')).toBeTruthy();
    expect(screen.getByPlaceholderText('Password')).toBeTruthy();
    expect(screen.getByText('Log In')).toBeTruthy();
  });

  it('calls onSubmit with form data', async () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);

    fireEvent.changeText(screen.getByPlaceholderText('Email'), 'test@example.com');
    fireEvent.changeText(screen.getByPlaceholderText('Password'), 'password123');
    fireEvent.press(screen.getByText('Log In'));

    await waitFor(() => {
      expect(mockOnSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123',
      });
    });
  });

  it('displays validation errors', async () => {
    render(<LoginForm onSubmit={mockOnSubmit} />);

    fireEvent.press(screen.getByText('Log In'));

    await waitFor(() => {
      expect(screen.getByText('Email is required')).toBeTruthy();
      expect(screen.getByText('Password is required')).toBeTruthy();
    });

    expect(mockOnSubmit).not.toHaveBeenCalled();
  });

  it('disables submit button while loading', () => {
    render(<LoginForm onSubmit={mockOnSubmit} isLoading />);

    const button = screen.getByText('Logging in...');
    expect(button).toBeDisabled();
  });
});
```

### BDD/Acceptance (Cucumber.js) {#bddacceptance-cucumberjs-rn}

#### Configuration

```javascript
// cucumber.js

module.exports = {
  default: {
    require: ['features/step_definitions/**/*.ts', 'features/support/**/*.ts'],
    requireModule: ['ts-node/register'],
    format: ['progress-bar', 'html:reports/cucumber.html'],
    formatOptions: { snippetInterface: 'async-await' },
    publishQuiet: true,
  },
};
```

```typescript
// features/support/world.ts

import { setWorldConstructor, World, IWorldOptions } from '@cucumber/cucumber';
import { by, device, element, expect as detoxExpect } from 'detox';

export interface CustomWorld extends World {
  device: typeof device;
  element: typeof element;
  by: typeof by;
  expect: typeof detoxExpect;
}

class CustomWorldImpl extends World implements CustomWorld {
  device = device;
  element = element;
  by = by;
  expect = detoxExpect;

  constructor(options: IWorldOptions) {
    super(options);
  }
}

setWorldConstructor(CustomWorldImpl);
```

#### Feature Files

```gherkin
# features/authentication/login.feature

Feature: Mobile Login
    As a mobile app user
    I want to log in to my account
    So that I can access my data

    Background:
        Given the app is launched

    @smoke @mobile
    Scenario: Successful login
        When I enter "user@example.com" in the email field
        And I enter "SecurePass123!" in the password field
        And I tap the login button
        Then I should see the dashboard screen
        And I should see "Welcome back"

    @mobile
    Scenario: Failed login with invalid credentials
        When I enter "user@example.com" in the email field
        And I enter "wrongpassword" in the password field
        And I tap the login button
        Then I should see an error alert "Invalid credentials"

    @mobile
    Scenario: Biometric login
        Given I have biometric authentication enabled
        When I tap the biometric login button
        And I authenticate with biometrics
        Then I should see the dashboard screen
```

#### Step Definitions

```typescript
// features/step_definitions/mobile.steps.ts

import { Given, When, Then } from '@cucumber/cucumber';
import { CustomWorld } from '../support/world';

Given('the app is launched', async function (this: CustomWorld) {
  await this.device.launchApp({ newInstance: true });
});

When('I enter {string} in the email field', async function (this: CustomWorld, email: string) {
  await this.element(this.by.id('email-input')).typeText(email);
});

When('I enter {string} in the password field', async function (this: CustomWorld, password: string) {
  await this.element(this.by.id('password-input')).typeText(password);
});

When('I tap the login button', async function (this: CustomWorld) {
  await this.element(this.by.id('login-button')).tap();
});

Then('I should see the dashboard screen', async function (this: CustomWorld) {
  await this.expect(this.element(this.by.id('dashboard-screen'))).toBeVisible();
});

Then('I should see {string}', async function (this: CustomWorld, text: string) {
  await this.expect(this.element(this.by.text(text))).toBeVisible();
});

Then('I should see an error alert {string}', async function (this: CustomWorld, message: string) {
  await this.expect(this.element(this.by.text(message))).toBeVisible();
});

Given('I have biometric authentication enabled', async function (this: CustomWorld) {
  // Mock biometric availability
  await this.device.setBiometricEnrollment(true);
});

When('I tap the biometric login button', async function (this: CustomWorld) {
  await this.element(this.by.id('biometric-button')).tap();
});

When('I authenticate with biometrics', async function (this: CustomWorld) {
  await this.device.matchBiometric();
});
```

### E2E (Detox)

#### Configuration

```javascript
// .detoxrc.js

module.exports = {
  testRunner: {
    args: {
      $0: 'jest',
      config: 'e2e/jest.config.js',
    },
    jest: {
      setupTimeout: 120000,
    },
  },
  apps: {
    'ios.debug': {
      type: 'ios.app',
      binaryPath: 'ios/build/Build/Products/Debug-iphonesimulator/MyApp.app',
      build: 'xcodebuild -workspace ios/MyApp.xcworkspace -scheme MyApp -configuration Debug -sdk iphonesimulator -derivedDataPath ios/build',
    },
    'ios.release': {
      type: 'ios.app',
      binaryPath: 'ios/build/Build/Products/Release-iphonesimulator/MyApp.app',
      build: 'xcodebuild -workspace ios/MyApp.xcworkspace -scheme MyApp -configuration Release -sdk iphonesimulator -derivedDataPath ios/build',
    },
    'android.debug': {
      type: 'android.apk',
      binaryPath: 'android/app/build/outputs/apk/debug/app-debug.apk',
      build: 'cd android && ./gradlew assembleDebug assembleAndroidTest -DtestBuildType=debug',
    },
    'android.release': {
      type: 'android.apk',
      binaryPath: 'android/app/build/outputs/apk/release/app-release.apk',
      build: 'cd android && ./gradlew assembleRelease assembleAndroidTest -DtestBuildType=release',
    },
  },
  devices: {
    simulator: {
      type: 'ios.simulator',
      device: {
        type: 'iPhone 15',
      },
    },
    emulator: {
      type: 'android.emulator',
      device: {
        avdName: 'Pixel_5_API_33',
      },
    },
  },
  configurations: {
    'ios.sim.debug': {
      device: 'simulator',
      app: 'ios.debug',
    },
    'ios.sim.release': {
      device: 'simulator',
      app: 'ios.release',
    },
    'android.emu.debug': {
      device: 'emulator',
      app: 'android.debug',
    },
    'android.emu.release': {
      device: 'emulator',
      app: 'android.release',
    },
  },
};
```

#### E2E Test Examples

```typescript
// e2e/authentication.test.ts

import { by, device, element, expect } from 'detox';

describe('Authentication', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  describe('Login', () => {
    it('should allow user to log in with valid credentials', async () => {
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();

      await expect(element(by.id('dashboard-screen'))).toBeVisible();
      await expect(element(by.text('Welcome back'))).toBeVisible();
    });

    it('should show error for invalid credentials', async () => {
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('wrongpassword');
      await element(by.id('login-button')).tap();

      await expect(element(by.text('Invalid credentials'))).toBeVisible();
    });

    it('should validate required fields', async () => {
      await element(by.id('login-button')).tap();

      await expect(element(by.text('Email is required'))).toBeVisible();
      await expect(element(by.text('Password is required'))).toBeVisible();
    });

    it('should handle keyboard dismiss', async () => {
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).tap();

      // Dismiss keyboard
      await device.pressBack();

      await expect(element(by.id('login-button'))).toBeVisible();
    });
  });

  describe('Biometric Authentication', () => {
    beforeEach(async () => {
      await device.setBiometricEnrollment(true);
    });

    it('should allow biometric login when enabled', async () => {
      await element(by.id('biometric-button')).tap();
      await device.matchBiometric();

      await expect(element(by.id('dashboard-screen'))).toBeVisible();
    });

    it('should handle biometric failure', async () => {
      await element(by.id('biometric-button')).tap();
      await device.unmatchBiometric();

      await expect(element(by.text('Biometric authentication failed'))).toBeVisible();
    });
  });

  describe('Logout', () => {
    beforeEach(async () => {
      // Login first
      await element(by.id('email-input')).typeText('test@example.com');
      await element(by.id('password-input')).typeText('password123');
      await element(by.id('login-button')).tap();
      await expect(element(by.id('dashboard-screen'))).toBeVisible();
    });

    it('should log out user successfully', async () => {
      await element(by.id('menu-button')).tap();
      await element(by.text('Logout')).tap();

      await expect(element(by.id('login-screen'))).toBeVisible();
    });
  });
});
```

---

## Node.js Stack

### Unit Tests (Vitest) {#unit-tests-vitest-node}

#### Configuration

```typescript
// vitest.config.ts

import { defineConfig } from 'vitest/config';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [tsconfigPaths()],
  test: {
    globals: true,
    environment: 'node',
    setupFiles: ['./tests/setup.ts'],
    include: ['**/*.{test,spec}.ts'],
    exclude: ['node_modules', 'dist', 'e2e'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
    },
    mockReset: true,
    restoreMocks: true,
  },
});
```

#### Unit Test Examples

```typescript
// src/services/__tests__/user.service.test.ts

import { describe, it, expect, vi, beforeEach } from 'vitest';
import { UserService } from '../user.service';
import { prisma } from '@/lib/prisma';

vi.mock('@/lib/prisma', () => ({
  prisma: {
    user: {
      create: vi.fn(),
      findUnique: vi.fn(),
      findMany: vi.fn(),
    },
  },
}));

describe('UserService', () => {
  let service: UserService;

  beforeEach(() => {
    service = new UserService();
    vi.clearAllMocks();
  });

  describe('createUser', () => {
    it('creates user with valid data', async () => {
      const userData = {
        email: 'test@example.com',
        name: 'Test User',
        password: 'hashedPassword',
      };

      vi.mocked(prisma.user.create).mockResolvedValue({
        id: '1',
        ...userData,
        createdAt: new Date(),
        updatedAt: new Date(),
      });

      const result = await service.createUser(userData);

      expect(prisma.user.create).toHaveBeenCalledWith({
        data: userData,
      });
      expect(result.email).toBe('test@example.com');
    });

    it('throws error for duplicate email', async () => {
      vi.mocked(prisma.user.create).mockRejectedValue(
        new Error('Unique constraint failed on the fields: (`email`)')
      );

      await expect(
        service.createUser({
          email: 'existing@example.com',
          name: 'Test',
          password: 'pass',
        })
      ).rejects.toThrow('User with this email already exists');
    });
  });
});
```

### BDD/Acceptance (Cucumber.js) {#bddacceptance-cucumberjs-node}

#### Feature Files

```gherkin
# features/api/users.feature

Feature: User API
    As an API consumer
    I want to manage users via REST API
    So that I can integrate with external systems

    Background:
        Given the API is running
        And I am authenticated as admin

    @api
    Scenario: Create a new user
        When I send a POST request to "/api/users" with:
            """
            {
                "email": "newuser@example.com",
                "name": "New User",
                "password": "SecurePass123!"
            }
            """
        Then the response status should be 201
        And the response should contain "email" with value "newuser@example.com"

    @api
    Scenario: Get user by ID
        Given a user exists with email "existing@example.com"
        When I send a GET request to "/api/users/{userId}"
        Then the response status should be 200
        And the response should contain "email" with value "existing@example.com"

    @api
    Scenario: List all users with pagination
        Given 25 users exist
        When I send a GET request to "/api/users?page=1&limit=10"
        Then the response status should be 200
        And the response should contain 10 users
        And the response should have pagination metadata
```

#### Step Definitions

```typescript
// features/step_definitions/api.steps.ts

import { Given, When, Then, Before, After } from '@cucumber/cucumber';
import { expect } from 'chai';
import supertest from 'supertest';
import { app } from '@/app';

let request: supertest.SuperTest<supertest.Test>;
let response: supertest.Response;
let authToken: string;
let createdUserId: string;

Before(async function () {
  request = supertest(app);
});

Given('the API is running', function () {
  // API is started by supertest
});

Given('I am authenticated as admin', async function () {
  const res = await request
    .post('/api/auth/login')
    .send({ email: 'admin@example.com', password: 'adminpass' });
  authToken = res.body.token;
});

Given('a user exists with email {string}', async function (email: string) {
  const res = await request
    .post('/api/users')
    .set('Authorization', `Bearer ${authToken}`)
    .send({ email, name: 'Test User', password: 'TestPass123!' });
  createdUserId = res.body.id;
});

Given('{int} users exist', async function (count: number) {
  for (let i = 0; i < count; i++) {
    await request
      .post('/api/users')
      .set('Authorization', `Bearer ${authToken}`)
      .send({
        email: `user${i}@example.com`,
        name: `User ${i}`,
        password: 'TestPass123!',
      });
  }
});

When('I send a {word} request to {string}', async function (method: string, endpoint: string) {
  const url = endpoint.replace('{userId}', createdUserId);
  response = await (request as any)[method.toLowerCase()](url)
    .set('Authorization', `Bearer ${authToken}`);
});

When('I send a {word} request to {string} with:', async function (
  method: string,
  endpoint: string,
  body: string
) {
  response = await (request as any)[method.toLowerCase()](endpoint)
    .set('Authorization', `Bearer ${authToken}`)
    .set('Content-Type', 'application/json')
    .send(JSON.parse(body));
});

Then('the response status should be {int}', function (status: number) {
  expect(response.status).to.equal(status);
});

Then('the response should contain {string} with value {string}', function (
  key: string,
  value: string
) {
  expect(response.body[key]).to.equal(value);
});

Then('the response should contain {int} users', function (count: number) {
  expect(response.body.data).to.have.lengthOf(count);
});

Then('the response should have pagination metadata', function () {
  expect(response.body).to.have.property('meta');
  expect(response.body.meta).to.have.property('page');
  expect(response.body.meta).to.have.property('limit');
  expect(response.body.meta).to.have.property('total');
});
```

### E2E (Cypress)

#### Configuration

**IMPORTANT:** Use Chrome for Cypress tests. Configure `browser: 'chrome'` or use the `--browser chrome` CLI flag.

```typescript
// cypress.config.ts

import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
    supportFile: 'cypress/support/e2e.ts',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: false,
    screenshotOnRunFailure: true,
    retries: {
      runMode: 2,
      openMode: 0,
    },
    // Use Chrome browser (CRITICAL)
    browser: 'chrome',
  },
  component: {
    devServer: {
      framework: 'react',
      bundler: 'vite',
    },
  },
});
```

```bash
# Run Cypress with Chrome (recommended)
npx cypress run --browser chrome

# Run Cypress with Chrome from environment variable
npx cypress run --browser $CHROME_PATH

# Open Cypress in Chrome for interactive testing
npx cypress open --browser chrome
```

```typescript
// cypress/support/e2e.ts

import './commands';

Cypress.on('uncaught:exception', () => {
  // Prevent Cypress from failing tests on uncaught exceptions
  return false;
});
```

```typescript
// cypress/support/commands.ts

declare global {
  namespace Cypress {
    interface Chainable {
      login(email: string, password: string): Chainable<void>;
      logout(): Chainable<void>;
      apiLogin(email: string, password: string): Chainable<string>;
    }
  }
}

Cypress.Commands.add('login', (email: string, password: string) => {
  cy.visit('/login');
  cy.get('[data-testid="email-input"]').type(email);
  cy.get('[data-testid="password-input"]').type(password);
  cy.get('[data-testid="login-button"]').click();
  cy.url().should('include', '/dashboard');
});

Cypress.Commands.add('logout', () => {
  cy.get('[data-testid="menu-button"]').click();
  cy.get('[data-testid="logout-button"]').click();
  cy.url().should('include', '/login');
});

Cypress.Commands.add('apiLogin', (email: string, password: string) => {
  return cy.request({
    method: 'POST',
    url: '/api/auth/login',
    body: { email, password },
  }).then((response) => {
    window.localStorage.setItem('token', response.body.token);
    return response.body.token;
  });
});

export {};
```

#### E2E Test Examples

```typescript
// cypress/e2e/authentication.cy.ts

describe('Authentication', () => {
  describe('Login', () => {
    beforeEach(() => {
      cy.visit('/login');
    });

    it('allows user to log in with valid credentials', () => {
      cy.get('[data-testid="email-input"]').type('user@example.com');
      cy.get('[data-testid="password-input"]').type('password123');
      cy.get('[data-testid="login-button"]').click();

      cy.url().should('include', '/dashboard');
      cy.contains('Welcome back').should('be.visible');
    });

    it('shows error for invalid credentials', () => {
      cy.get('[data-testid="email-input"]').type('user@example.com');
      cy.get('[data-testid="password-input"]').type('wrongpassword');
      cy.get('[data-testid="login-button"]').click();

      cy.get('[role="alert"]').should('contain', 'Invalid credentials');
      cy.url().should('include', '/login');
    });

    it('validates required fields', () => {
      cy.get('[data-testid="login-button"]').click();

      cy.contains('Email is required').should('be.visible');
      cy.contains('Password is required').should('be.visible');
    });

    it('redirects authenticated users', () => {
      cy.apiLogin('user@example.com', 'password123');
      cy.visit('/login');

      cy.url().should('include', '/dashboard');
    });
  });

  describe('Logout', () => {
    beforeEach(() => {
      cy.login('user@example.com', 'password123');
    });

    it('logs out user successfully', () => {
      cy.logout();

      cy.url().should('include', '/login');
      cy.get('[data-testid="login-button"]').should('be.visible');
    });
  });

  describe('Registration', () => {
    beforeEach(() => {
      cy.visit('/register');
    });

    it('allows new user to register', () => {
      cy.get('[data-testid="name-input"]').type('New User');
      cy.get('[data-testid="email-input"]').type('newuser@example.com');
      cy.get('[data-testid="password-input"]').type('SecurePass123!');
      cy.get('[data-testid="confirm-password-input"]').type('SecurePass123!');
      cy.get('[data-testid="register-button"]').click();

      cy.url().should('include', '/dashboard');
      cy.contains('Welcome, New User').should('be.visible');
    });

    it('validates password confirmation', () => {
      cy.get('[data-testid="name-input"]').type('New User');
      cy.get('[data-testid="email-input"]').type('newuser@example.com');
      cy.get('[data-testid="password-input"]').type('SecurePass123!');
      cy.get('[data-testid="confirm-password-input"]').type('DifferentPass!');
      cy.get('[data-testid="register-button"]').click();

      cy.contains('Passwords do not match').should('be.visible');
    });
  });
});
```

#### API Testing with Cypress

```typescript
// cypress/e2e/api/users.cy.ts

describe('Users API', () => {
  let authToken: string;

  beforeEach(() => {
    cy.apiLogin('admin@example.com', 'adminpass').then((token) => {
      authToken = token;
    });
  });

  describe('GET /api/users', () => {
    it('returns list of users', () => {
      cy.request({
        method: 'GET',
        url: '/api/users',
        headers: { Authorization: `Bearer ${authToken}` },
      }).then((response) => {
        expect(response.status).to.eq(200);
        expect(response.body).to.have.property('data');
        expect(response.body.data).to.be.an('array');
      });
    });

    it('supports pagination', () => {
      cy.request({
        method: 'GET',
        url: '/api/users?page=1&limit=10',
        headers: { Authorization: `Bearer ${authToken}` },
      }).then((response) => {
        expect(response.status).to.eq(200);
        expect(response.body.meta.page).to.eq(1);
        expect(response.body.meta.limit).to.eq(10);
      });
    });
  });

  describe('POST /api/users', () => {
    it('creates a new user', () => {
      cy.request({
        method: 'POST',
        url: '/api/users',
        headers: { Authorization: `Bearer ${authToken}` },
        body: {
          email: 'newuser@example.com',
          name: 'New User',
          password: 'SecurePass123!',
        },
      }).then((response) => {
        expect(response.status).to.eq(201);
        expect(response.body.email).to.eq('newuser@example.com');
      });
    });

    it('validates required fields', () => {
      cy.request({
        method: 'POST',
        url: '/api/users',
        headers: { Authorization: `Bearer ${authToken}` },
        body: {},
        failOnStatusCode: false,
      }).then((response) => {
        expect(response.status).to.eq(400);
        expect(response.body.errors).to.have.property('email');
        expect(response.body.errors).to.have.property('name');
        expect(response.body.errors).to.have.property('password');
      });
    });
  });
});
```
