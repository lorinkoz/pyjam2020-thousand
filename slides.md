name: title
class: middle

.left[![Logo of Pyjamas](images/logo.png)]

# .high[A thousand Djangoes within]<br/><small>or Django multi-tenant</small>

---

## Syntax error

Django.emph[es] &rightarrow; Django

-   Django.emph[s]
-   Django.emph[es]

--

<br/>

# A thousand Django.emph[es] within

---

## Hi, I am...

.left-column-66[

#### Lorenzo Peña

-   Proud citizen of Holguín, Cuba.
-   Django developer for 11 years.
-   Capable of falling asleep basically anywhere.

]
.right-column-33[![Photo of myself](images/lorinkoz.jpg)]

---

class: middle
layout: false

## .blue[How to fit a thousand Djangoes in a single Django?]

--

# .green[Why does it matter?]

---

layout: true

## A bedtime story of fitting Djangoes

---

--

-   Once upon a time customer .red[**red**] had a problem.

--

-   Our brave hero developed a solution in Django.

--

-   But then, customers .blue[**blue**], .green[**green**] and .yellow[**yellow**] showed up with the same problem.

--

.left-column-66[.box[🤔 What should our hero do?]]
.right-column-33[.right[![Screenshot of Townscaper with a tiny red house](images/problem-solved.png)]]

---

.center[![Screenshot of Townscaper with tiny houses](images/single-tenancy.png)]

---

.center[![Screenshot of Townscaper with an apartment building](images/multi-tenancy.png)]

---

layout: true

## Multi-tenancy

---

.left-column-66[

-   Software architecture
-   Single instance of software
-   Serves multiple tenants

]

.right-column-33[.right[![Diagram of multi-tenanacy](images/diagram-multi-tenancy.png)]]

--

![Dropbox logo](images/dropbox-logo.png)
![Shopify logo](images/shopify-logo.png)
![Slack logo](images/slack-logo.png)
![Wordpress logo](images/wordpress-logo.png)

---

layout: true

## Tenants

---

.left-column-66[

-   Isolated spaces
-   Users with specific privileges interact

]
.right-column-33[
.right[![Diagram of tenants](images/diagram-tenant.png)]
]

---

.left-column-33[
.center[![Dropbox logo](images/dropbox-logo.png) Accounts]
]

.left-column-33[
.center[![Slack logo](images/slack-logo.png) Workspaces]
]

.left-column-33[
.center[![Wordpress logo](images/wordpress-logo.png) Blogs]
]

.left-column-33[
.center[![Discord logo](images/discord-logo.png) Servers]
]

.left-column-33[
.center[![Shopify logo](images/shopify-logo.png) Stores]
]

.left-column-33[
.center[![StackExchange logo](images/stackexchange-logo.png) Sites]
]

---

layout: false

.box[`echo "Awesome!"`<br/>`python3 manage.py add-multi-tenancy --asap`]

--

.right[![Meme of Boromir "One does not simply walk into Mordor" about multi-tenancy](images/boromir-mordor-meme-multi-tenancy.png)]

---

layout: false
class: middle

# A thousand Djangoes within

--

-   The active tenant
-   Three architectural choices
-   The scope of everything else

---

layout: true

## The active tenant

---

---

.left-column[.center[![StackExchange logo](images/stackexchange-logo.png)]]

--

.right-column[

```python
Question.objects.all()
```

]

--

<br/>

.center[**Ok, but where?**]

<br/>

--

.left-column-33[.center[![StackOverflow logo](images/stackexchange-stackoverflow-logo.png)]]
.left-column-33[.center[![ServerFault logo](images/stackexchange-serverfault-logo.png)]]
.left-column-33[.center[![SuperUser logo](images/stackexchange-superuser-logo.png)]]

---

.box[New concept: **the active tenant** 😎]

--

.left-column[

-   Database access
-   URL reversing
-   Admin site
-   Cache
    ]

.right-column[

-   Channels (websockets)
-   Management commands
-   Celery tasks
-   File storage
    ]

--

.warning[👀 Even outside the request / response cycle]

---

Django has a couple APIs we're likely familiar with:

|             |                          |                      |
| ----------- | ------------------------ | -------------------- |
| 🌐 Timezone | `get_current_timezone()` | `activate(timezone)` |
| 🈸 Language | `get_current_language()` | `activate(language)` |

--

We could also have:

|           |                        |                    |
| --------- | ---------------------- | ------------------ |
| 🏠 Tenant | `get_current_tenant()` | `activate(tenant)` |

---

```python
# Drop-in replacement for threading.locals that works with asyncio
from asgiref.local import Local

_active = Local()

def get_current_tenant():
    return getattr(_active, "tenant", None)

def activate(tenant):
    _active.tenant = tenant
```

--

.warning[⚠️ The use of globals is generally discouraged]

---

##### Two important questions to ask:

--

.box[What is the 🧪 **type** of a tenant object?]

--

.box[What happens if for some operation there is<br/>😶 **no active tenant**?]

---

class: middle
layout: false

# Three architectural choices

--

name: architectural-choices

1. Users and tenants
2. Database architecture
3. Tenant routing

---

layout: true

## Users and tenants

---

---

Users exist **outside** the context of tenants:

.left-column-66[
.left[![Diagram of users outside tenants](images/diagram-users-out-tenants.png)]
]

--

.right-column-33[
.center[![WordPress logo](images/wordpress-logo.png)]
.center[![Shopify logo](images/shopify-logo.png)]
.center[![Discord logo](images/discord-logo.png)]
]

---

Users exist **within** the context of tenants:

.left-column-66[
.left[![Diagram of users inside tenants](images/diagram-users-in-tenants.png)]
]

--

.right-column-33[
.center[![Slack logo](images/slack-logo.png)]
]

---

Users exist **as** tenants:

.left-column-66[
.left[![Diagram of users equalling tenants](images/diagram-users-equal-tenants.png)]
]

--

.right-column-33[
.center[![Gmail logo](images/gmail-logo.png)]
.center[![Dropbox logo](images/dropbox-logo.png)]
]

---

.box[🙋‍♀️ Which one to pick?]

---

class: middle
layout: false

# Database architecture

---

## Database architecture

**Isolated:**<br/>Multiple databases, one per tenant

**Shared:**<br/>One database, tenant column on entry-level tables

**Semi-isolated:**<br/>One database, one schema per tenant (PostgreSQL)

---

layout: true

## Isolated databases

---

.center[![Diagram of isolated tenants](images/diagram-isolated.png)]

---

Multi-database configuration in Django settings:

```python
DATABASES = {
    "default": {...},
    "tenant1": {...},
    "tenant2": {...},
    "tenant3": {...},
    ...
}
```

---

Queries need to rely on the active tenant:

```python
tenant = get_current_tenant()
active_db = `get_database_for_tenant(tenant)`

question = Question(...)
question.save(`using=active_db`)

Question.objects.`using(active_db)`.filter(...)
Question.objects.`db_manager(active_db)`.do_something(...)
```

---

Scope could be offloaded to **database routers**:

```python
class IsolatedTenantsDatabaseRouter:

    def `db_for_read`(self, model, **hints):
        tenant = get_current_tenant()
        return `get_database_for_tenant(tenant)`

    def `db_for_write`(self, model, **hints):
        tenant = get_current_tenant()
        return `get_database_for_tenant(tenant)`
```

---

**.green[The good]**

-   Optimized for isolation.

--

**.red[The bad]**

-   No relations across databases.
-   Adding tenants require reconfiguring the project.

---

.center[![Scrooge McDuck in a pile of money](images/mcduck-gold.png)]

---

layout: true

## Shared database

---

.center[![Diagram of shared database](images/diagram-shared.png)]

---

.emph[Entry-level, tenant-specific models] require a pointer to the tenant they belong to:

```python
class Question(models.Model):

    tenant = models.ForeignKey("Site", ...)

    # Rest of the model
    ...

```

---

Queries need to rely on the active tenant:

```python
tenant = get_current_tenant()

Question.objects.create(`tenant=tenant`, ...)

Question.objects.filter(`tenant=tenant`, ...)
Answer.objects.filter(`question__tenant=tenant`, ...)
some_user.questions.filter(`tenant=tenant`, ...)
```

---

Tenant field could be automatically **assigned** via:

-   Default value for the field (a callable).
-   Custom field (`ForeignKey` subclass) with a `pre_save` hook.
-   `pre_save` signal on relevant models.

--

Tenant scope could be automatically **queried** via:

-   Custom manager.
-   Custom queries.

---

**.green[The good]**

-   Optimized for scalability.

--

**.red[The bad]**

-   Data isolation takes extra development effort.

---

.left-column[![Meme of brain not letting you sleep on scoped queries](images/brain-sleep-meme-scopes.png)]

--

.right-column[

-   Bookmark all your tenant scoped queries.
-   Test each one of them.
-   Test that you tested each one of them.
    ]

--

.right-column[
.box[🧸 Tests are a soft pillow]
]

---

layout: true

## Semi-isolated database

---

.right[![Diagram of semi-isolated tenants](images/diagram-semi-isolated.png)]

---

.left-column-66[Relies on PostgreSQL schemas.ref[1] to isolate tenants within a single database.
{{content}}]
.right-column-33[.right[![Diagram of semi-isolated tenants](images/diagram-semi-isolated.png)]]

.bottom[
.footnote[.ref[1] https://www.postgresql.org/docs/current/ddl-schemas.html]
]

--

##### Schemas:

-   Layer between database and tables.
-   Equivalent to namespaces.
-   Searchable through the **search path**.

---

Your queries remain unchanged:

```python
Question.objects.all()
Question.objects.create(...)
```

--

.warning[👀 Increased technical challenge somewhere else]

---

##### This is where:

--

-   Custom database backend to set the search path.

--

-   Custom `migrate` command to operate with schemas.

--

-   Custom database router with `allow_migrate`.

---

**.green[The good]**

-   Optimized for isolation with increased scalability.

--

**.red[The bad]**

-   Extra effort to understand and control how schemas interact.

---

.center[![Meme of crazy lady and cat about schemas and migrations](images/cat-lady-meme-schemas.png)]

---

layout: true

## Which one is the best?

---

---

.warning[🔥 Neither!]

--

-   Pros and cons in all of them.
-   Depends on your specific use case.

---

class: middle
layout: false

# Tenant routing

---

layout: true

## Tenant routing

---

.box[🤔 How to activate a tenant from a request?]

--

##### In general

-   Inferred from the user
-   Stored in the session
-   Specified in the headers

---

.box[🤔 How to activate a tenant from a request?]

##### Specifically from the URL

|                     |                                        |
| ------------------- | -------------------------------------- |
| Via domain          | .emph[`tenant1`]`.example.com/view/`   |
| Via subfolder       | `example.com/`.emph[`tenant1`]`/view/` |
| Via query parameter | `example.com/view/?t=`.emph[`tenant1`] |

---

.box[🚦 Middleware are the perfect place to do it!]

--

```python
def TenantFromSessionMiddleware(get_response):
    def middleware(request):
        tenant = `get_tenant_from_session(request.session)`
        if tenant and not get_current_tenant():
            activate(tenant)
        return get_response(request)
    return middleware
```

---

Multiple retrieval methods can be implemented:

|           |                                              |
| --------- | -------------------------------------------- |
| user      | `get_tenant_from_user(request.user)`         |
| headers   | `get_tenant_from_headers(request.headers)`   |
| domain    | `get_tenant_from_domain(request.get_host())` |
| subfolder | `get_tenant_from_subfolder(request.path)`    |
| params    | `get_tenant_from_params(request.GET)`        |

--

.box[💡 Multiple middleware can be chained]

---

layout: true

## Reversing tenant-aware URLs

---

.box[🙋‍♀️ How to reverse URLs so that the tenant is included?]

--

For some cases it's simply not possible:

|     |                          |
| --- | ------------------------ |
| ❌  | Inferred from the user   |
| ❌  | Stored in the session    |
| ❌  | Specified in the headers |

---

The only possible way is when the tenant is inferred from the URL itself:

|     |                     |                                        |
| --- | ------------------- | -------------------------------------- |
| ✔️  | Via domain          | .emph[`tenant1`]`.example.com/view/`   |
| ✔️  | Via subfolder       | `example.com/`.emph[`tenant1`]`/view/` |
| ✔️  | Via query parameter | `example.com/view/?t=`.emph[`tenant1`] |

--

.box[💡 Requires tweaking the URL reversing process]

---

layout: false

## Tenant-specific URLConfs

--

.box[💡 Have different URLConfs based on the active tenant]

--

```python
def URLConfFromTenantMiddleware(get_response):
    def middleware(request):
        tenant = get_current_tenant()
*       request.urlconf = get_urlconf_from_tenant(tenant)
        return get_response(request)
    return middleware
```

---

template: architectural-choices

---

class: middle
layout: false

# The scope of everything else

--

1. Management commands
2. File storage
3. Cache
4. Celery tasks
5. Channels (websockets)

---

## Management commands

For new commands, a tenant argument can be included.

```shell
python3 manage.py do_something_in_tenant `-t tenant1`
```

--

For existing, non tenant-aware commands, a .emph[special command wrapper] must be defined.

```shell
python3 manage.py `tenant_wrap loaddata some_fixture` -t tenant1
```

---

## File storage

Takes a custom file storage that organizes files per tenant.

--

##### Higher security contexts:

-   Generate pre-signed URLs.
-   Use a proxy view.

---

## Cache

Needs a tenant-specific cache key function:

```python
# settings.py
CACHES = {
    "default": {
        ...
        `"KEY_FUNCTION": "myproject.cache.get_key_from_tenant",`
    }
}

# myproject/cache.py
def get_key_from_tenant(key, key_prefix, version):
    tenant = get_current_tenant()
    return `"{}:{}:{}:{}".format(tenant, key_prefix, version, key)`
```

---

## Celery tasks

The task must expect a tenant to activate as part of its parameters:

```python
@app.task(bind=True)
def some_celery_task(self, `tenant_id`, ...):
    tenant = `get_tenant_from_id(tenant_id)`
    activate(tenant)
    ...
```

---

## Channels (websockets)

-   Requires custom middleware to activate tenant from request.
-   Requires naming your consumer groups including the tenant (for proper tenant groups isolation)

---

## For everything that was not covered

--

.box[💡 Principles are generally extensible]

---

class: middle
layout: false

# Now, some of the packages

---

layout: true

## Available packages

---

##### Shared database

-   [citusdata/django-multitenant](https://github.com/citusdata/django-multitenant)
-   [raphaelm/django-scopes](https://github.com/raphaelm/django-scopes)

##### Semi-isolated database

-   [bernardopires/django-tenant-schemas](https://github.com/bernardopires/django-tenant-schemas)
-   [tomturner/django-tenants](https://github.com/tomturner/django-tenants)

.bottom[
.footnote[More context in https://djangopackages.org/grids/g/multi-tenancy/]
]

---

layout: false

## And that's it!

##### We can keep in touch here:

|         |                                                    |
| ------- | -------------------------------------------------- |
| Twitter | [@lorinkoz](https://twitter.com/lorinkoz)          |
| GitHub  | [github.com/lorinkoz](https://github.com/lorinkoz) |
| Email   | [lorinkoz@gmail.com](mailto:lorinkoz@gmail.com)    |

--

.right[![Figurines used to represent users with sunglasses](images/figurines.png)]

---

template: title
