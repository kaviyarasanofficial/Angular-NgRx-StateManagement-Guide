# 🚀 NgRx State Management Setup in Angular (Standalone App)

This guide will walk you through setting up **NgRx** in a **new Angular standalone application** using best practices — **without `NgModule`**, using the latest Angular features like `ApplicationConfig`.

---

## ✅ Quick Setup Commands

First, install the NgRx schematics and core libraries:

```bash
ng add @ngrx/schematics@latest

# Install individual packages
ng add @ngrx/store --save
ng add @ngrx/effects --save
ng add @ngrx/store-devtools --save
ng add @ngrx/entity --save
```

---

## 🛠️ Configure State in `app.config.ts`

```ts
import { ApplicationConfig, provideZoneChangeDetection, isDevMode } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { routes } from './app.routes';
import { provideStore } from '@ngrx/store';
import { provideStoreDevtools } from '@ngrx/store-devtools';
import { provideEffects } from '@ngrx/effects';

import { upscMainScholarlyReducer } from './store/upscMainScholarly/upsc-main-scholarly.reducer';
import { UpscMainScholarlyEffects } from './store/upscMainScholarly/upsc-main-scholarly.effects';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideHttpClient(),
    provideStore({ upscMainScholarly: upscMainScholarlyReducer }),
    provideEffects([UpscMainScholarlyEffects]),
    provideStoreDevtools({ maxAge: 25, logOnly: !isDevMode() }),
  ],
};
```

---

## 📁 Folder Structure

```
src/app/store/upscMainScholarly/
  ├── upsc-main-scholarly.actions.ts
  ├── upsc-main-scholarly.reducer.ts
  ├── upsc-main-scholarly.selectors.ts
  └── upsc-main-scholarly.effects.ts  (optional)
```

---

## 🧩 Generate Files

```bash
# Step 1: Create the folder
mkdir -p src/app/store/upscMainScholarly

# Step 2: Generate actions
ng generate action store/upscMainScholarly/upscMainScholarly --flat

# Step 3: Generate reducer
ng generate reducer store/upscMainScholarly/upscMainScholarly --flat --skip-tests

# Step 4: Generate selectors
ng generate selector store/upscMainScholarly/upscMainScholarly --flat

# Step 5: Generate effects (manually create if needed)
ng generate effect store/upscMainScholarly/upscMainScholarly --flat --skip-tests
```

---

## 🎯 Example Usage

### Component Method (Trigger Action)

```ts
addMainsScholarly() {
  const req = {
    name: this.upscMainScholarlyPapers.name,
    key: this.upscMainScholarlyPapers.key,
    isTypesOfSection: this.upscMainScholarlyPapers.isTypesOfSection,
    parentId: this.upscMainScholarlyPapers.parentId,
  };
  this.scholarlyStore.dispatch(UpscMainScholarlyActions.addNewUpscMainScholarly({ data: req }));
}
```

### `upsc-main-scholarly.actions.ts`

```ts
import { createActionGroup, props } from '@ngrx/store';

export const UpscMainScholarlyActions = createActionGroup({
  source: 'UpscMainScholarly',
  events: {
    'Add New UpscMainScholarly': props<{ data: any }>(),
  },
});
```

### `upsc-main-scholarly.reducer.ts`

```ts
import { createReducer, on } from '@ngrx/store';
import { UpscMainScholarlyActions } from './upsc-main-scholarly.actions';

export interface MainScholarlyPapersState {
  data: any[];
}

export const initialState: MainScholarlyPapersState = {
  data: [],
};

export const upscMainScholarlyReducer = createReducer(
  initialState,
  on(UpscMainScholarlyActions.addNewUpscMainScholarly, (state, { data }) => ({
    ...state,
    data: [...state.data, data],
  }))
);
```

### `upsc-main-scholarly.selectors.ts`

```ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { MainScholarlyPapersState } from './upsc-main-scholarly.reducer';

export const selectMainScholarlyPapersFeature = createFeatureSelector<MainScholarlyPapersState>('upscMainScholarly');

export const selectMainScholarlyPapers = createSelector(
  selectMainScholarlyPapersFeature,
  (state) => state.data
);
```

---

## 👀 Viewing in Component

### `view-mains-scholarly.component.ts`

```ts
export class ViewMainsScholarlyComponent {
  scholarlyPapers$: Observable<any[]>;

  constructor(private scholarlyStore: Store<MainScholarlyPapersState>) {
    this.scholarlyPapers$ = this.scholarlyStore.select(selectMainScholarlyPapers);
  }
}
```

### `view-mains-scholarly.component.html`

```html
<tr *ngFor="let scholarlyPaper of scholarlyPapers$ | async">
  <td>{{ scholarlyPaper.name }}</td>
  <td>{{ scholarlyPaper.key }}</td>
  <td>{{ scholarlyPaper.isTypesOfSection }}</td>
  <td>{{ scholarlyPaper.parentId }}</td>
</tr>
```

---

## 🙌 Final Notes

- This guide uses modern Angular features (standalone APIs).
- Designed for simplicity, flexibility, and debugging ease.
- Logs are included to help trace state flows (`console.log` in selectors and reducers).

---

🔗 Happy State Managing with NgRx!