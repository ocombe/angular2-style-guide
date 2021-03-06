# Angular 2 Style Guide

The purpose of the following style guide is to present a set of best practices and style guidelines for the development of Angular 2 applications.
If you are looking for an opinionated style guide for syntax, conventions, and structuring Angular 2 applications, then you can step in!

![Angular 2 Style Guide](/assets/logo.png)

**Disclaimer: The document is in alpha version which means that some the guidelines will change and new will be added.**

You are welcome to join the discussion of the best [practices here](https://github.com/mgechev/angular2-style-guide/issues).

The guidelines described below are based on:

1. Angular 2 [source code](https://github.com/angular/angular).
2. Suggestions by [Miško Hevery](https://github.com/mhevery) from his technical review of "[Switching to Angular 2](https://www.packtpub.com/web-development/switching-angular-2)".
3. My own development experience working in a team on large-scale Angular 2 application.
4. [John Papa's AngularJS 1.x style guide](https://github.com/johnpapa/angular-styleguide), for being consistent with the directory structure and testing across the different major versions of the framework.

# Table of Contents

1. [Directory Structure](#directory-structure)
2. [Modules](#modules)
3. [Directives and Components](#directives-and-components)
4. [Services and Dependency Injection](#services-and-dependency-injection)
5. [Pipes](#pipes)
6. [Routing](#routing)
7. [Forms](#forms)
8. [Testing](#testing)
9. [Change Detection](#change-detection)
11. [TypeScript specific practices](#typescript-specific-practices)
12. [ES2015 and ES2016 specific practices](#es2015-and-es2016-specific-practices)
13. [ES5 specific practices](#es5-specific-practices)
14. [License](#license)

# Directory Structure

* Create folders named for the feature they represent. When a folder grows to contain more than 7 files, start to consider creating a folder for them. Your threshold may be different, so adjust as needed.

  *Why?*: A developer can locate the code, identify what each file represents at a glance, the structure is flat as can be, and there is no repetitive nor redundant names.

  *Why?*: When there are a lot of files (10+) locating them is easier with a consistent folder structures and more difficult in flat structures.

  ```
  ├── app
  │   ├── about
  │   │   └── components
  │   │       ├── about.e2e.ts
  │   │       ├── about.ts
  │   │       └── about.spec.ts
  │   ├── master
  │   │   └── components
  │   │       ├── master.css
  │   │       ├── master.e2e.ts
  │   │       ├── master.html
  │   │       ├── master.ts
  │   │       └── master.spec.ts
  │   ├── assets
  │   │   ├── img
  │   │   │   └── smile.png
  │   │   └── main.css
  │   ├── home
  │   │   └── components
  │   │       ├── home.css
  │   │       ├── home.ts
  │   │       └── home.spec.ts
  │   ├── shared
  │   │   └── services
  │   │       ├── name_list.ts
  │   │       └── name_list.spec.ts
  │   ├── main.ts
  │   └── index.html
  ```

# Modules

* Keep the modules self-contained and coherent. Each module should have a [single reason to change](https://en.wikipedia.org/wiki/Single_responsibility_principle).

  *Why?*: This way the modules will be more reusable, and thus composable.

* Keep each code unit (component, directive, service, pipe, etc.) into a separate file. If the unit uses other internal for the given module logic you can keep it in the same module, without exporting it.

  *Why?*: The definitions will be easier to find simply by looking at the directory structure.

  *Why?*: Do not export private APIs because you need to manage them and keep them consistent for the end users.

* Name the modules with `lower_snake_case`.

  *Why?*: Using lower case will not cause problem across platforms with different case sensitivity.

  *Why?*: By using only English letters, numbers and underscore (`_`) the file names will be consistent across platforms.

  ### Example

  **CORRECT**
  ```
  sg_tooltip.ts
  sg_user_service.ts
  ```

# Directives and Components

* Use attributes as selectors for your directives.

  *Why?*: There could be many directives per element, which makes it more suitable to use attributes as opposed to elements.

* Use custom prefix for the selector of your directives (for instance below is used the prefix `sg` from **S**tyle **G**uide).

  *Why?*: This way you will be able to prevent name collisions.

  ### Example

  **WRONG**

  ```ts
  @Directive({
    selector: '[tooltip]'
  })
  class BootstrapTooltipDir {}

  @Directive({
    selector: '[tooltip]'
  })
  class CustomTooltipDir {}

  @Component({
    selector: 'app',
    template: `...`,
    directives: [CustomTooltip, BootstrapTooltip]
  })
  class AppCmp {}
  ```

  **CORRECT**

  ```ts
  @Directive({
    selector: '[bs-tooltip]'
  })
  class BsBootstrapTooltipDir {}

  @Directive({
    selector: '[my-tooltip]'
  })
  class MyCustomTooltipDir {}

  @Component({
    selector: 'sg-app',
    template: `...`,
    directives: [CustomTooltip, BootstrapTooltip]
  })
  class SgAppCmp {}
  ```

* Name directives' controllers with `Dir` suffix and components' controllers with `Cmp` suffix. The name of any directive or component should be formed following the rule `CustomPrefix` + `BasicDescription` + `Dir` or `Cmp`.

  ### Example
  **CORRECT**
  ```ts

  @Directive({
    selector: '[sg-tooltip]`
  })
  class SgTooltipDir {}

  @Component({
    selector: 'sg-button',
    template: `...`
  })
  class SgButtonCmp {}
  ```

  *Why?*: When any code unit is imported the consumer will know how to use it based on its name, i.e. `SgButtonCmp` means that:

    - This is a controller of a component.
    - The component should be used as an element.
    - Its selector is `sg-button`.

* Use `@HostListener` and `@HostBinding` instead of the `host` property of the `@Directive` and `@Component` decorators:

  *Why?*: The name of the property, or method name associated to `@HostBinding` or respectively `@HostListener` should be modified only in a single place - in the directive's controller. In contrast if you use `host` you need to modify both the property declaration inside the controller, and the metadata associated to the directive.

  *Why?*: The metadata declaration attached to the directive is shorter and thus more readable.

  ### Example

  **WRONG**
  ```ts
  @Directive({
    selector: '[sg-sample'],
    host: {
      '(mouseenter)': 'onMouseEnter()',
      'role': 'button'
    }
  })
  class SgSampleDir {
    button;
    onMouseEnter() {...}
  }
  ```

  **CORRECT**
  ```ts
  @Directive({
    selector: '[sg-sample]'
  })
  class SgSampleDir {
    @HostBinding('role') button;
    @HostListener('mouseenter') onMouseEnter() {...}
  }
  ```

* Use element selectors for components.

  *Why?*: There could be only a single component per element.

  *Why?*: Components are the actual elements in our applications, compared to directives which only augment the elements.

  ### Example
  **WRONG**
  ```ts
  @Component({
    selector: '[sg-button]',
    template: `...`
  })
  class SgButtonCmp {}
  ```

  **CORRECT**
  ```ts
  @Component({
    selector: 'sg-button',
    template: `...`
  })
  class SgButtonCmp {}
  ```


* Keep the components as simple and coherent as possible but not too simple.

  *Why?*: Simple coherent components are easier to reason about, more reusable and composable.

  *Why?*: Components which are too primitive may lead to scattering and harder management of the user interface of our applications.

* Keep the components' templates as lean as possible and inline them inside of the `@Component` decorator.

  *Why?*: Keeping the components' templates short implies simple, composable components.

  *Why?*: Keeping the template next to the component's controller makes it easier to lookup the component's structure and/or modify it.

* Extract the more complex and bigger templates, longer than 15 lines of code, into a separate file and put them next to their controllers' definition.

  *Why?*: In case a big and complex template is inlined in the component metadata it may shift the focus from the component's logic defined within the controller.

* Use `@Input` and `@Output` instead of the `inputs` and `outputs` properties of the `@Directive` and `@Component` decorators:

  *Why?*: The name of the property, or event name associated to `@Input` or respectively `@Output` should be modified only on a single place.

  *Why?*: The metadata declaration attached to the directive is shorter and thus more readable.

  ### Example

  **WRONG**
  ```ts
  @Component({
    selector: 'sg-button',
    template: `...`,
    inputs: [
      'label'
    ],
    outputs: [
      'change'
    ]
  })
  class SgButtonCmp {
    change = new EventEmitter<any>();
    label: string;
  }
  ```

  **CORRECT**
  ```ts
  @Component({
    selector: 'sg-button',
    template: `...`
  })
  class SgButtonCmp {
    @Output()
    change = new EventEmitter<any>();
    @Input()
    label: string;
  }
  ```
* Do not rename inputs and outputs.

  *Why?*: May lead to confusion when the output or the input properties of a given directive are named a given way but exported differently as a public API.

  ### Example

  **WRONG**
  ```ts
  @Component({
    selector: 'sg-button',
    template: `...`
  })
  class SgButtonCmp {
    @Output('changeEvent') change = new EventEmitter<any>();
    @Input('labelAttribute') label: string;
  }
  /*
   * Need to be consumed the following way:
   *
   * <sg-button labelAttribute="foobar" (changeEvent)="doStuff()">
   * </sg-button>
   */
  ```

  **CORRECT**
  ```ts
  @Component({
    selector: 'sg-button',
    template: `...`
  })
  class SgButtonCmp {
    @Output() change = new EventEmitter<any>();
    @Input() label: string;
  }
  /*
   * Need to be consumed the following way, which is much more straightforward:
   *
   * <sg-button label="foobar" (change)="doStuff()">
   * </sg-button>
   */
  ```

* Prefer inputs over the `@Attribute` parameter decorator.

  *Why?*: The input creates one-way binding which means that we can bind it to an expression and its value gets automatically updated. We can inject a property with `@Attribute` to a controller's constructor and get its value a single time.

  ### Example

  **WRONG**
  ```ts
  @Component({
    selector: 'sg-button',
    template: `...`
  })
  class SgButtonCmp {
    label: string;
    constructor(@Attribute('label') label) {
      this.label = label;
    }
  }
  ```

  **CORRECT**
  ```ts
  @Component({
    selector: 'sg-button',
    template: `...`
  })
  class SgButtonCmp {
    @Input() label: string;
  }
  ```

* Detach components and directives which are not visible from the view in order to prevent the change detection running over them.
* Do not inject native elements to the controller's constructors.

  *Why?*: This way the application will get tightly coupled to the platform and thus won't be able to run independently from it. For instance, a web application injecting native DOM elements won't be able to run in WebWorker, nor be rendered on the server-side.

  ### Example
  **WRONG**
  ```ts
  @Component({
    selector: 'sg-text-field',
    template: `<input type="text">`
  })
  class SgTextFieldCmp {
    value: string;
    constructor(el: ElementRef) {
      el.nativeElement.querySelector('input[type="text"]')
        .onchange = () => this.value = el.value;
    }
  }
  ```

  **CORRECT**
  ```ts
  import {NgModel} from 'angular2/common';

  @Component({
    directives: [NgModel],
    selector: 'sg-text-field',
    template: `<input [(ngModel)]="value" type="text">`
  })
  class SgTextFieldCmp {
    value: string;
  }
  ```

* Do not manipulate element referenced within the template.

  *Why?*: This way the application will get tightly coupled to the platform and thus won't be able to run independently from it. For instance, a web application injecting native DOM elements won't be able to run in WebWorker, neither be rendered on the server-side.

  **WRONG**
  ```ts
  @Component({
    selector: 'sg-items-list',
    template: `
      <input type="text" #textInput>
      <button (click)="add(textInput.value)">Add</button>
    `
  })
  class SgItemListCmp {
    values: string[] = [];
    add(val) {
      this.values.push(val);
    }
  }
  ```

  **CORRECT**
  ```ts
  import {NgModel} from 'angular2/common';

  @Component({
    directives: [NgModel],
    selector: 'sg-items-list',
    template: `
      <input type="text" [(ngModel)]="value">
      <button (click)="add()">Add</button>
    `
  })
  class SgItemListCmp {
    value: string;
    values: string[] = [];
    add() {
      this.values.push(this.value);
    }
  }
  ```

# Services and Dependency Injection

* Limit the usage of `forwardRef`.

  *Why?*: The usage of `forwardRef` indicates either a cyclic dependency or inconsistency in the services' declaration (i.e. the dependent service is declared before its dependency). In both cases there is usually a better approach to be used.

* Do not declare global providers in the `bootstrap` call, use a top-level component instead.

  *Why?*: The providers registered in the `bootstrap` method are meant for overriding existing providers rather than declaring dependencies.

# Pipes

* Use impure pipes only when they need to hold state.

  *Why?*: The change detection can optimize pure pipes since they hold the [referential transparency](https://en.wikipedia.org/wiki/Referential_transparency) property.

# Routing

* Name the routes the same way the components associated with them are called.

  *Why?*: This way there is only a single name associated to a given route. This avoids confusion in case a given route is called in a different way to the components associated with it.

# Forms

* Do not use expression for name of a control.

  *Why?*: When the name of the control is dynamically generated it will be hard to reference it inside of the controller associated to the form, and may lead to confusion.

  ### Example
  **WRONG**
  ```html
  <form>
    <input [ngControl]="foobar" type="text">
  </form>
  ```
  ```ts
  @Component(...)
  class SampleCmp {
    foobar = 'foo';
  }
  ```
  **CORRECT**
  ```html
  <form>
    <input ngControl="foo" type="text">
  </form>
  ```
  ```ts
  @Component(...)
  class SampleCmp {}
  ```

# Testing

* Use [Jasmine](https://jasmine.github.io/) for unit testing.

  *Why?*: It is well supported and popular in the JavaScript community.

  *Why?*: It is widely popular in the Angular community. Jasmine is used for testing both AngularJS 1.x and Angular 2.

  *Why?*: It has up-to-date type definitions so you can have great development experience using TypeScript.

* Use Karma as test runner.

* Place your test files side-by-side with your client code.

  *Why?*: The tests are easier to find since they are always next to the code they are testing.

  *Why?*: When you update source code it is easier to go update the tests at the same time.

  *Why?*: The code may act as documentation of the tested component.

* Name the test file the following way `NAME_OF_THE_TESTED_UNIT.spec.EXT`:

  ### Example:
  **CORRECT**
  ```
  about.ts
  about.spec.ts
  ```

* Place integration tests and tests that cover multiple code units into a separate `tests` directory in your project's root.

# Change detection

* Use `OnPush` change detection strategy for [pure/dumb components](http://teropa.info/blog/2015/10/18/refactoring-angular-apps-to-components.html#toward-smart-and-dumb-components) that accepts as input immutable data.

  *Why?*: Angular will optimize the performance of your application dramatically by not performing change detection over the entire sub tree with root the given pure component.

# TypeScript specific practices

* Use tslint for linting your code.

  *Why?*: In big teams, code following the same practices is easier to read and understand.

* Be as explicit in the type definitions as possible (i.e. use `any` as rarely as possible).

  *Why?*: `any` makes TypeScript behaves like a dynamic language. This way we gave up all the benefits we get from static typing such as better IDE/text editor support and compile-time type-checking.

* Use "Façade modules" for exporting the public members of your application/library and hiding the implementation details.

  ### Example
  ```ts
  export {NgClass} from './directives/ng_class';
  export {NgFor} from './directives/ng_for';
  export {NgIf} from './directives/ng_if';
  export {NgStyle} from './directives/ng_style';
  export {NgSwitch, NgSwitchWhen, NgSwitchDefault} from './directives/ng_switch';
  export * from './directives/observable_list_diff';
  export {CORE_DIRECTIVES} from './directives/core_directives';
  ```

## TypeScript dependency injection

* Use the `@Injectable()` decorator instead of explicitly declaring the dependencies using `@Inject(TOKEN)`.

  *Why?*: Using `@Injectable()` the TypeScript compiler will emit the required type metadata, which makes the code using `@Inject()` unnecessary verbose and less readable.

## TSLint

* Use TSLint for linting your JavaScript and be sure to customize the TSLint options file and include in source control. See the [TSLint docs](https://github.com/palantir/tslint) for details on the options.

  *Why?*: Provides a first alert prior to committing any code to source control.

  *Why?*: Provides consistency across your team.

  ```json
  {
    "rules": {
      "class-name": true,
      "curly": false,
      "eofline": true,
      "indent": "spaces",
      "max-line-length": [true, 140],
      "member-ordering": [true,
        "public-before-private",
        "static-before-instance",
        "variables-before-functions"
      ],
      "no-arg": true,
      "no-construct": true,
      "no-duplicate-key": true,
      "no-duplicate-variable": true,
      "no-empty": true,
      "no-eval": true,
      "no-trailing-comma": true,
      "no-trailing-whitespace": true,
      "no-unused-expression": true,
      "no-unused-variable": true,
      "no-unreachable": true,
      "no-use-before-declare": true,
      "one-line": [true,
        "check-open-brace",
        "check-catch",
        "check-else",
        "check-whitespace"
      ],
      "quotemark": [true, "single"],
      "semicolon": true,
      "triple-equals": true,
      "variable-name": false
    }
  }
  ```

# ES2015 and ES2016 specific practices



# ES5 specific practices

* Use the internal JavaScript DLS provided by Angular 2 for annotating constructor functions.

  *Why?*: This syntax is simpler, more readable and closer to the original TypeScript version of the code.

  *Why?*: The DLS uses function expressions which are closer to the non-hoisted ES2015 classes.

  ### Example

  **WRONG**
  ```js
  function Component() {
    //...
  }
  Component.annotations = [
    new ng.core.ComponentMetadata({
      //...
    }),
    new ng.router.RouteConfig([
      //...
    ])
  ];
  ```

  **CORRECT**
  ```ts
  var Component = ng.
    Component({
      //...
    })
    .RouteConfig([
      //...
    ])
    .Class({
      constructor: functions () {
        //...
      }
    });
  ```

## JSHint

* Use JSHint for linting your JavaScript and be sure to customize the JSHint options file and include in source control. See the [JSHint](http://www.jshint.com/docs/) docs for details on the options.

  *Why?*: Provides a first alert prior to committing any code to source control.

  *Why?*: Provides consistency across your team.

  ```json
  {
      "bitwise": true,
      "camelcase": true,
      "curly": true,
      "eqeqeq": true,
      "es3": false,
      "forin": true,
      "freeze": true,
      "immed": true,
      "indent": 4,
      "latedef": "nofunc",
      "newcap": true,
      "noarg": true,
      "noempty": true,
      "nonbsp": true,
      "nonew": true,
      "plusplus": false,
      "quotmark": "single",
      "undef": true,
      "unused": false,
      "strict": false,
      "maxparams": 10,
      "maxdepth": 5,
      "maxstatements": 40,
      "maxcomplexity": 8,
      "maxlen": 120,

      "asi": false,
      "boss": false,
      "debug": false,
      "eqnull": true,
      "esnext": false,
      "evil": false,
      "expr": false,
      "funcscope": false,
      "globalstrict": false,
      "iterator": false,
      "lastsemic": false,
      "laxbreak": false,
      "laxcomma": false,
      "loopfunc": true,
      "maxerr": false,
      "moz": false,
      "multistr": false,
      "notypeof": false,
      "proto": false,
      "scripturl": false,
      "shadow": false,
      "sub": true,
      "supernew": false,
      "validthis": false,
      "noyield": false,

      "browser": true,
      "node": true,

      "globals": {
          "angular": false,
          "ng": false
      }
  }
  ```

## JSCS

* Use JSCS for checking your coding styles your JavaScript and be sure to customize the JSCS options file and include in source control. See the [JSCS docs](http://jscs.info/rules) for details on the options.

  *Why?*: Provides a first alert prior to committing any code to source control.

  *Why?*: Provides consistency across your team.

  ```json
  {
      "excludeFiles": ["node_modules/**", "bower_components/**"],

      "requireCurlyBraces": [
          "if",
          "else",
          "for",
          "while",
          "do",
          "try",
          "catch"
      ],
      "requireOperatorBeforeLineBreak": true,
      "requireCamelCaseOrUpperCaseIdentifiers": true,
      "maximumLineLength": {
        "value": 100,
        "allowComments": true,
        "allowRegex": true
      },
      "validateIndentation": 4,
      "validateQuoteMarks": "'",

      "disallowMultipleLineStrings": true,
      "disallowMixedSpacesAndTabs": true,
      "disallowTrailingWhitespace": true,
      "disallowSpaceAfterPrefixUnaryOperators": true,
      "disallowMultipleVarDecl": null,

      "requireSpaceAfterKeywords": [
        "if",
        "else",
        "for",
        "while",
        "do",
        "switch",
        "return",
        "try",
        "catch"
      ],
      "requireSpaceBeforeBinaryOperators": [
          "=", "+=", "-=", "*=", "/=", "%=", "<<=", ">>=", ">>>=",
          "&=", "|=", "^=", "+=",

          "+", "-", "*", "/", "%", "<<", ">>", ">>>", "&",
          "|", "^", "&&", "||", "===", "==", ">=",
          "<=", "<", ">", "!=", "!=="
      ],
      "requireSpaceAfterBinaryOperators": true,
      "requireSpacesInConditionalExpression": true,
      "requireSpaceBeforeBlockStatements": true,
      "requireLineFeedAtFileEnd": true,
      "disallowSpacesInsideObjectBrackets": "all",
      "disallowSpacesInsideArrayBrackets": "all",
      "disallowSpacesInsideParentheses": true,

      "jsDoc": {
          "checkAnnotations": true,
          "checkParamNames": true,
          "requireParamTypes": true,
          "checkReturnTypes": true,
          "checkTypes": true
      },

      "disallowMultipleLineBreaks": true,

      "disallowCommaBeforeLineBreak": null,
      "disallowDanglingUnderscores": null,
      "disallowEmptyBlocks": null,
      "disallowTrailingComma": null,
      "requireCommaBeforeLineBreak": null,
      "requireDotNotation": null,
      "requireMultipleVarDecl": null,
      "requireParenthesesAroundIIFE": true
  }
  ```

# License

MIT
