
# NgRx Scholarly Papers Setup (Angular)

This README walks you through the complete implementation of an NgRx store setup for managing scholarly papers in an Angular application using standalone APIs (without NgModules).

---

## 🔧 Installation

```bash
ng add @ngrx/schematics@latest

ng add @ngrx/store --save
ng add @ngrx/effects --save
ng add @ngrx/store-devtools --save
ng add @ngrx/entity --save
```

---

## 🛠️ App Configuration (app.config.ts)

```ts
import { ApplicationConfig, provideZoneChangeDetection, isDevMode } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { provideStore } from '@ngrx/store';
import { provideStoreDevtools } from '@ngrx/store-devtools';
import { provideEffects } from '@ngrx/effects';

import { routes } from './app.routes';
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
  ├── upsc-main-scholarly.effects.ts
```

---

## 🚀 Actions (upsc-main-scholarly.actions.ts)

```ts
import { createActionGroup, props, emptyProps } from '@ngrx/store';

export const UpscMainScholarlyActions = createActionGroup({
  source: 'UpscMainScholarly',
  events: {
    'Add New UpscMainScholarly': props<{ data: any }>(),
    'Load Scholarly Papers': emptyProps(),
    'Load Scholarly Papers Success': props<{ data: any[] }>(),
    'Load Scholarly Papers Failure': props<{ error: any }>(),
  }
});
```

---

## 🔁 Reducer (upsc-main-scholarly.reducer.ts)

```ts
import { createReducer, on } from '@ngrx/store';
import { UpscMainScholarlyActions } from './upsc-main-scholarly.actions';

export interface MainScholarlyPapersState {
  data: any[];
}

export const initialState: MainScholarlyPapersState = {
  data: []
};

export const upscMainScholarlyReducer = createReducer(
  initialState,
  on(UpscMainScholarlyActions.addNewUpscMainScholarly, (state, { data }) => ({
    ...state,
    data: [...state.data, data]
  })),
  on(UpscMainScholarlyActions.loadScholarlyPapersSuccess, (state, { data }) => ({
    ...state,
    data
  }))
);
```

---

## 📦 Effects (upsc-main-scholarly.effects.ts)

```ts
import { inject } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { switchMap, map, catchError } from 'rxjs/operators';
import { of } from 'rxjs';
import { ScholarlyService } from '../../services/scholarly.service';
import { UpscMainScholarlyActions } from './upsc-main-scholarly.actions';

export const UpscMainScholarlyEffects = createEffect(() => {
  const actions$ = inject(Actions);
  const scholarlyService = inject(ScholarlyService);

  return actions$.pipe(
    ofType(UpscMainScholarlyActions.loadScholarlyPapers),
    switchMap(() =>
      scholarlyService.getScholarlyPapers().pipe(
        map(data => UpscMainScholarlyActions.loadScholarlyPapersSuccess({ data })),
        catchError(error => of(UpscMainScholarlyActions.loadScholarlyPapersFailure({ error })))
      )
    )
  );
});
```

---

## 📡 Selector (upsc-main-scholarly.selectors.ts)

```ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { MainScholarlyPapersState } from './upsc-main-scholarly.reducer';

export const selectMainScholarlyPapersFeature = createFeatureSelector<MainScholarlyPapersState>('upscMainScholarly');

export const selectMainScholarlyPapers = createSelector(
  selectMainScholarlyPapersFeature,
  (state: MainScholarlyPapersState) => state.data
);
```

---

## 🧩 View Component (view-mains-scholarly.component.ts)

```ts
import { Component, OnInit } from '@angular/core';
import { Observable } from 'rxjs';
import { Store } from '@ngrx/store';
import { MainScholarlyPapersState } from '../../store/upscMainScholarly/upsc-main-scholarly.reducer';
import { selectMainScholarlyPapers } from '../../store/upscMainScholarly/upsc-main-scholarly.selectors';
import { UpscMainScholarlyActions } from '../../store/upscMainScholarly/upsc-main-scholarly.actions';
import { Actions, ofType } from '@ngrx/effects';

@Component({
  selector: 'app-view-mains-scholarly',
  templateUrl: './view-mains-scholarly.component.html'
})
export class ViewMainsScholarlyComponent implements OnInit {
  scholarlyPapers$: Observable<any[]>;

  constructor(
    private scholarlyStore: Store<MainScholarlyPapersState>,
    private actions$: Actions
  ) {
    this.scholarlyPapers$ = this.scholarlyStore.select(selectMainScholarlyPapers);
  }

  ngOnInit(): void {
    this.reloadData();

    this.actions$.pipe(ofType('[API] Add Success')).subscribe(() => {
      this.reloadData();
    });
  }

  reloadData() {
    this.scholarlyStore.dispatch(UpscMainScholarlyActions.loadScholarlyPapers());
  }
}
```

---

## 📝 View Template (view-mains-scholarly.component.html)

```html
<tr *ngFor="let scholarlyPaper of scholarlyPapers$ | async">
  <td>{{ scholarlyPaper.name }}</td>
  <td>{{ scholarlyPaper.key }}</td>
  <td>{{ scholarlyPaper.isTypesOfSection }}</td>
  <td>{{ scholarlyPaper.parentId }}</td>
</tr>
```

---

✅ This setup ensures:
- Clean folder structure
- Store state management for scholarly papers
- Automatic refresh on successful add
- Reload on page initialization
