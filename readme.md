
Naming Conventions
-------------------

Generally follow the angular2 style guide

https://angular.io/styleguide

Typescript files should named as "snake-case-name.type.ts" where type is the angular type: component, directive, pipe, etc.

    hero-edit.component.ts
    hero-search.component.ts
    hero.service.ts

The typescript standard for naming conventions is:

    export class MyPascalCaseClass {
        localCamelCaseField: string = '';

        constructor(private camelCaseParameter: string, localCamelCaseParameter: Hero) {

        }
        
        // note: some linters suggest requiring the "public" keyword, which is the default, we should decide
        somePublicCamelCaseMethod (camelCaseParameter: string) : boolean {
            return !!camcelCaseParameter;
        }

        private camelLocalMethod(): void {
            this.localCamelCaseField = 'field';
        }

        private autoBindingLocalMethod: (foo: string) => boolean = this.localCamelCaseField + foo === 'foobar';
    }

    // interfaces should begin with I
    export interface IHero {
        name: string;
        friends: IHero[];
    }


Discussion:
    A standard naming for components, such as
        object-action.component.ts.   i.e.,   
            store-search.component.ts
            store-edit.component.ts  (or edit-store, etc.)



Patterns
--------

Objects on the wire should be typed as an interface.  The server should put on the wire the exact data that is needed by the client 
    (we *may* at some put a code generator in the project that creates a typescript interface from a c# interface)

/* hero.ts */
export interface IHero {
    name: string;
    strength: number;
    superpower?: string;
}

/* hero-edit.component.ts */

    export class HeroEditComponent  {
        hero: Hero = <Hero> {};
        // ...
        this.service.get(id).subscribe(hero => this.hero = <Hero> hero);       
    } 

if the model has behavior that needs to be in a class, create an interface to represent the data that goes on the wire.  
    *consider* using "Object.assign" to move the data to the class (unit tests might help here)

    export interface HeroData {
        name: string;
    }
    export class Hero implements HeroData {
        name: string;
        isValid () :boolean {
            return !!this.name;
        }

        constructor (HeroData data) {
            this.name = 'some name';
            Object.assign(this, data);
        }

    }

Server API
----------

Consider a restful API

    /api/retail/stores  
        
        GET /api/retail/stores (get all stores)
        GET /api/retail/stores/123  (get store ID 123)
        POST /api/retail/stores   (create or update a store)
        PUT /api/retail/stores/123 (update store 123)
        DELETE /api/retail/stores/123  (delete store 123)  (never returns 404)
    
    /api/retail/storesearch
        GET /api/retail/storesearch   (get an empty search criteria)
        POST /api/retail/storesearch  (perform a search)

    /api/retail/storesearchlookups
        GET /api/lookups/searchlookups   (return a cacheable set of lookups for store search, aka 
                    {
                        locations: [{id: 1, value: 'Los Angeles'}, {id: 2, value 'Orange'}],
                        
    Each of these apis represents a controller on the server, and a service on the client.  The client service may subclass RestServiceBase to avoid writing any code.

    A few notes on restful APIs...

        Technically, PUT is for replacing a resource on the server, while POST is for saving a recourse.  People often map that to UPDATE and CREATE, while others
        just use POST for both.   It doesn't really matter, but we should have a standard. 

        DELETE should be idempotent, to use a twenty-five cent word that basically means that multiple DELETE calls with the same parameter should return the same thing.  
        In practice, this means that if the item being deleted can't be found, treat that as success (the item has been deleted, after all).  Obviously, the business
        can override this if it's important.

        API actions should almost never return null.  Either return the resource requested, or return a non-success error code (i.e., throw an exception).  The client
        app should almost never need to check for null (and shouldn't).

        It's usually worthwhile returning something from DELETE, usually either the ID or a boolean true.  It's not really necessary, it just cleans up certain responses.  We
        should have a standard.

Global Services
---------------

### CoreModalService

Inject CoreModalService to easily display dialogs

    this.modalService.ok('title', 'content');

    this.modalService.yesNo('title', 'Are you sure').then(b => b && doSomething());

The "partial" variants return a function, since that's often what you want, especially when working with observables or promises.  For example,

    subject.catch(() => this.modalService.ok('error', 'error'));

    // can be cleaned up as

    subject.catch(this.modalService.okPartial('error', 'error'));

    // or even cleaner

    successDialog = this.modalService.okPartial('Good', 'good dog');

    errorDialog = this.modalService.okPartial('Bad', 'bad dog'));

    someSource.subsribe(successDialog, errorDialog);


### Standard date pipes

Instead of littering the code with literal date strings that may need to be localized later, use the standard e3 pipes
    
    <span>{{ myDateValue | e3Date }}</span><span>{{ myDateTime | e3DateTime }}</span>

Try to avoid
    
    <span>{{ myDateValue | date:"MM/dd/yyyy" }}</span>

### RestServiceBase

Use this as your base class to inherit a standard set of restful services, along with date mapping, error logging, etc.
Inject the RestServiceParameterFactory so that your class isn't tightly coupled to the constructor of the base class

    const parameters = {
        url: '/my/url/to/the/server',
        idField: 'fooId'
    };

    @Injectable()
    export class LookupsService extends RestServiceBase<any> {
        constructor (factory: RestServiceParameterFactory) {
            super(factory.getParams(parameters));
        }
    }

