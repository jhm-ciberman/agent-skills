---
name: php-docblock-writing
description: "Review and fix the quality of PHP docblocks (`/** */`) and `//` comments on changed code. Use after coding a Laravel feature to keep the description a contract, cut tags that restate a typed signature, refine generics, copy parent docblocks verbatim, and stop docblock churn. Quality only - it does not hunt for correctness bugs."
---

# PHP Docblock Writing

You changed PHP. Pass over the `/** */` and `//` in the diff, fix the docs, touch no
behavior. Match `laravel/framework`. Keep it terse: one line per member by default,
Laravel docs are shorter than you think.

Mood is free: "Get the user." or "Gets the user.", both fine. Never churn a doc to flip it.

## Phase 0 - Gather the diff

`git diff @{upstream}...HEAD` (or `main...HEAD`, `HEAD~1`, `HEAD`). Scope: every
`/** */` and `//` the diff touched, plus the docs on any member it touched.

## Phase 1 - Review

Check each docblock against the rules below: the how leaked into the description, a
tag that restates the signature, a missing refinement the signature can't express, an
`@inheritDoc`, a class docblock that restates the name, an undocumented member, an
overpromise, a stable doc reworded for nothing.

## Phase 2 - Apply

Fix each. Skip anything that changes behavior, and say so.

## Phase 3 - Re-read your own work

For every doc you wrote: is it a contract, or did I describe the body? Does each tag
add what the signature can't? Did I overpromise? On an override, did I copy the parent
verbatim (not `@inheritDoc`)? Did I reword a stable doc? Fix what fails, then summarize
by file.

> Zero diff is fine. It means the code was already documented well.

---

## A docblock is a contract.

It says what a caller relies on, never how the code
works. The test for every sentence: would deleting it change what a *caller* needs to
know? No -> it's the how, cut it.

```php
// BAD - the how
/**
 * Set the paid_at column to the current time.
 */
public function markPaid(): void

// GOOD - the effect
/**
 * Mark the order as paid.
 */
public function markPaid(): void
```

## Tags carry the type

A tag earns its place only when it says something the native signature can't, or its
description adds context beyond the type and name.

As a general rule, don't add extra params documentation if it's already intuitive from the signature or argument names.

```php
// BAD - the descriptions restate type + name. Pint keeps them (they have text), so YOU cut them.
/**
 * Normalize a product SKU.
 *
 * @param  string  $sku  The SKU.
 * @return string  The normalized SKU.
 */
public static function normalizeSku(string $sku): string

// GOOD
/**
 * Normalize a product SKU.
 */
public static function normalizeSku(string $sku): string

// GOOD - states something that the signature can't
/**
 * Normalize a product SKU.
 *
 * @param  string  $sku  The SKU to normalize, which needs to be uppercase and trimmed.
 */
public static function normalizeSku(string $sku): string

// GOOD - @param refines a type the signature can't; the typed params get no tag
/**
 * Find the open order for the given reference.
 *
 * @param  array<string>|null  $allowedStatuses  Order statuses the caller accepts.
 *
 * @throws OrderNotFoundException If no open order matches the reference.
 */
public function findOpenOrder(Customer $customer, string $reference, ?array $allowedStatuses = null): Order
```

- `@param`: add it for an untyped param, or to refine (`array<string>`, `class-string<Enum>`). Skip a typed param unless a description adds real context.
- `@return`: refine a bare `array`/`Collection`/relation into its element type (`Comment[]`). Skip when precise (`string`, a typed model).
- `@var`: untyped or generic property. Skip a typed one (`bool $active`).
- Null semantics go in the description, not a tag: "Get the label, or null if none applies." on a plain `?string`.
- Pint auto-strips only a tag with no description whose type matches the signature exactly. Everything else is on you.

## Maximize generic specificity

```php
/** @return Collection<int, Comment> */
/** @return HasMany<Comment, $this> */
/** @param  class-string<Enum>  $enum */
/** @param  array{id: int, name: string}  $row */
/** @use HasFactory<PostFactory> */
```

Prefer these over bare `array`/`Collection`. Use a method-local `@template` for
map/first/get transforms, and class-level `@template`/`@extends`/`@implements`/`@use`
on generic classes and trait-use lines.

## Tag order and `@throws`

Description, blank, `@template`, blank, `@param` + `@return` (no blank between), blank,
`@throws`. Document a `@throws` that is part of the contract, one per line with a short
"when". Skip framework and transitive throws (`\Throwable`, `\PDOException`).

Generally, only document a `@throws` when it's part of the domain knowledge.

```php
/**
 * Charge the order and mark it as paid.
 *
 * @throws PaymentFailedException If the payment provider declines the charge.
 * @throws OrderAlreadyPaidException If the order has already been paid.
 */
public function charge(Customer $customer, Order $order): Payment
```

## Coverage: members 100%, classes ~10%

- **Members**: document every one, methods (incl. `private`/`protected`), properties,
  constants, enum cases. The forgotten ones are properties, constants, enum cases.
- **Classes**: most get no class docblock. The name and base class already say it.

## Classes: skip unless the name can't say it

```php
// BAD - restates the name
/**
 * An order processor.
 */
class OrderProcessor

// GOOD - the -er name says it
class OrderProcessor
```

Worth a one-liner only when the name can't carry the contract: an interface or
abstract base, a non-obvious model lifecycle, an enum's role, "Thrown when X" on a
non-obvious exception, a custom Filament (or Nova) field. Plenty of interfaces and
exceptions still skip it. Never restate the class name, rename the class instead.

## Overrides: copy the parent verbatim, never `@inheritDoc`

```php
// BAD - banned in this codebase
/** @inheritDoc */
public function handle(): void

// GOOD - the parent's text, copied
/**
 * Handle the queued job.
 */
public function handle(): void
```

The real docblock lives at the topmost class, trait, or interface. Copy it verbatim on
every override, including its `@param`/`@return` when the signature is untyped. Never
specialize the wording to the subclass (`Dog::makeSound()` keeps `Animal`'s "Emit this
animal's sound").

## Enums: name the role, document every case

```php
/**
 * Defines the reason codes for error responses.
 */
enum ErrorReason: string
{
    /**
     * The user's email address has not been verified.
     */
    case EmailNotVerified = 'auth:email-not-verified';
}
```

Don't list the cases in the type description, it goes stale the moment you add one.

## `//` comments: WHY, never WHAT

```php
// BAD - restates the line
$order->save(); // save the order

// GOOD - non-obvious WHY, timeless
// Don't save it yet. The caller decides whether to persist.
```

Default to none. No "TODO"/"#123"/"now also". A lone `//` in an empty body is a
deliberate marker, keep it. Config files group settings with Laravel `|--- ---|` block
headers. No docblocks on `test_*` methods, stubs, or mocks.

## Don't add comments that someone that haven't read our chat would not understand.

Comments should be timeless.

```php
// BAD - references this plan of this coding session
$this->doThink(); // This will be changed in phase 2

// OK - only if totally necesary
$this->doThink(); // Prevents issue #123

// GOOD - timeless and adds value (the WHY, not the WHAT)
$this->doThink(); // This is a costly operation, so we only want to do it once.
```

## Remember again: A docblock is a contract.

```php
// BAD - this explains how in the phpdoc contract. It IS important. But is NOT contract.

/**
 * Renders a friendly HTML page for authorization errors that can't be redirected
 * back to the client (an unknown client or an unregistered redirect URI), instead
 * of Passport's raw OAuth JSON. The raw OAuth error stays visible on the page so a
 * partner integrating the flow can diagnose it.
 *
 * Bound over Passport's controller so the authorize route resolves to this one.
 */
class AuthorizationController extends BaseAuthorizationController
{
    /**
     * Authorize a client to access the user's account.
     */
    public function authorize(
        ServerRequestInterface $psrRequest,
        Request $request,
        ResponseInterface $psrResponse,
        AuthorizationViewResponse $viewResponse
    ): Response|AuthorizationViewResponse {
        try {
            return parent::authorize($psrRequest, $request, $psrResponse, $viewResponse);
        } catch (OAuthServerException $e) {
            if ($e->getResponse()->isRedirect()) {
                throw $e;
            }

            $previous = $e->getPrevious();

            return response()->view('errors.oauth-client', [
                'status' => $e->getResponse()->getStatusCode(),
                'error' => $previous instanceof LeagueException ? $previous->getErrorType() : 'invalid_request',
                'description' => $previous?->getMessage(),
                'hint' => $previous instanceof LeagueException ? $previous->getHint() : null,
            ], $e->getResponse()->getStatusCode());
        }
    }
}

// GOOD - moved to a regular implementation comment. (However, only add comments when they are really necessary, otherwise skip them.)

// This class is bound over Passport's controller so the authorize route resolves to this one.
class AuthorizationController extends BaseAuthorizationController
{
    /**
     * Authorize a client to access the user's account.
     */
    public function authorize(
        ServerRequestInterface $psrRequest,
        Request $request,
        ResponseInterface $psrResponse,
        AuthorizationViewResponse $viewResponse
    ): Response|AuthorizationViewResponse {
        try {
            return parent::authorize($psrRequest, $request, $psrResponse, $viewResponse);
        } catch (OAuthServerException $e) {
            if ($e->getResponse()->isRedirect()) {
                throw $e; // We don't want to mess with Passport redirects
            }

            // We render a friendly HTML page for authorization errors that can't be redirected
            // back to the client (an unknown client or an unregistered redirect URI), instead
            // of Passport's raw OAuth JSON. The raw OAuth error stays visible on the page so a
            // partner integrating the flow can diagnose it.

            $previous = $e->getPrevious();

            return response()->view('errors.oauth-client', [
                'status' => $e->getResponse()->getStatusCode(),
                'error' => $previous instanceof LeagueException ? $previous->getErrorType() : 'invalid_request',
                'description' => $previous?->getMessage(),
                'hint' => $previous instanceof LeagueException ? $previous->getHint() : null,
            ], $e->getResponse()->getStatusCode());
        }
    }
}
```

## Don't churn, don't invent

On a refactor, touch a docblock only if the **contract** changed (or it was wrong).
Implementation changed but the contract didn't? Leave it byte for byte. And never
invent a contract you can't verify: read more files until you understand it, a
confident wrong doc is worse than none. If you still can't, leave it and flag it.
