# The Angular Model - ngx-model
by [@tomastrajan](https://twitter.com/tomastrajan)

[![npm](https://img.shields.io/npm/v/ngx-model.svg)](https://www.npmjs.com/package/ngx-model) [![npm](https://img.shields.io/npm/l/ngx-model.svg)](https://github.com/tomastrajan/ngx-model/blob/master/LICENSE) [![npm](https://img.shields.io/npm/dm/ngx-model.svg)](https://www.npmjs.com/package/ngx-model) [![Build Status](https://travis-ci.org/tomastrajan/ngx-model.svg?branch=master)](https://travis-ci.org/tomastrajan/ngx-model) [![Twitter Follow](https://img.shields.io/twitter/follow/tomastrajan.svg?style=social&label=Follow)](https://twitter.com/tomastrajan)

Simple state management with minimalistic API, one way data flow, 
multiple model support and immutable data exposed as RxJS Observable.


## Documentation
                           
* [Demo & Documentation](http://tomastrajan.github.io/angular-model-pattern-example/) 
* [Blog Post](https://medium.com/@tomastrajan/model-pattern-for-angular-state-management-6cb4f0bfed87) 
* [Changelog](https://github.com/tomastrajan/ngx-model/blob/master/CHANGELOG.md)

![ngx-model dataflow diagram](https://raw.githubusercontent.com/tomastrajan/angular-model-pattern-example/master/src/assets/model_graph.png "ngx-model dataflow diagram")

## Getting started

1. Install `ngx-model`
    
    ```
    npm install --save ngx-model
    ``` 
 
    or
    
    ```
    yarn add ngx-model
    ``` 

2. Import and use `NgxModelModule` in you `AppModule` (or `CoreModule`)
    ```
    import { NgxModelModule } from 'ngx-model';
        
    @NgModule({
      imports: [
        NgxModelModule
      ]
    })
    export class CoreModule {}
    
    ``` 

3. Import and use `Model` and `ModelFactory` in your own services.
    ```
    import { Injectable } from '@angular/core';
    import { ModelFactory, Model } from 'ngx-model';
        
    @Injectable()
    export class TodosService {
            
      private model: Model<Todo[]>;
      
      todos$: Observable<Todo[]>;
            
      constructor(private modelFactory: ModelFactory<Todo[]>) {
        this.model = this.modelFactory.create([]); // create model and pass initial data
        this.todos$ = this.model.data$; // expose model data as named public property
      }
        
      toggleTodo(id: string) {
        // retrieve raw model data
        const todos = this.model.get();
            
        // mutate model data
        todos.forEach(t => {
          if (t.id === id) {
            t.done = !t.done;
          }
        });
            
        // set new model data (after mutation)
        this.model.set(todos);
      }
        
    }
    ```

4. Use service in your component. Import and inject service into components constructor.
Subscribe to services data in template `todosService.todos$ | async` 
or explicitly `this.todosService.todos$.subscribe(todos => { /* ... */ })`

    ```
    import { Component, OnInit, OnDestroy } from '@angular/core';
    import { Subject } from 'rxjs/Subject';
    
    import { TodosService, Todo } from './todos.service';
    
    @Component({
      selector: 'ngx-model-todos',
      templateUrl: `
        /* ... */
        <h1>Todos ({{count}})</h1>
        <ul>
          <!-- template subscription to todos using async pipe -->
          <li *ngFor="let todo of todosService.todos$ | async" (click)="onTodoClick(todo)">
            {{todo.name}}
          </li>
        </ul>
      `,
    })
    export class TodosComponent implements OnInit, OnDestroy {
    
      private unsubscribe$: Subject<void> = new Subject<void>();
      
      count: number;
     
      constructor(public todosService: TodosService) {}
    
      ngOnInit() {
        // explicit subscription to todos to get count
        this.todosService.todos
          .takeUntil(this.unsubscribe$) // declarative unsubscription
          .subscribe(todos => this.count = todos.length);
      }
      
      ngOnDestroy(): void {
        // for declarative unsubscription
        this.unsubscribe$.next();
        this.unsubscribe$.complete();
      }
    
      onTodoClick(todo: Todo) {
        this.todosService.toggleTodo(todo.id);
      }
    
    }

    ```

## Relationship to Angular Model Pattern

This is a library version of [Angular Model Pattern](https://tomastrajan.github.io/angular-model-pattern-example).
All the original examples and documentation are still valid. The only difference is that
you can install `ngx-model` from npm instead of having to copy model pattern
implementation to your project manually.

Check out the [Blog Post](https://medium.com/@tomastrajan/model-pattern-for-angular-state-management-6cb4f0bfed87) and 
[Advanced Usage Patterns](https://tomastrajan.github.io/angular-model-pattern-example#/advanced) 
for more how-tos and examples.
