# types-redux-orm

Typescript types for [Redux Orm](https://github.com/tommikaikkonen/redux-orm)

## Important

It's an alpha version of types I currently use in one of my projects. They will be updated as I proceed with the project. No backward compatibility guaranteed.

## Roadmap

01/13/18 - Beta release
02/03/18 - Stable release and merge into [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped)

## Install

```
npm install types-redux-orm
```

Modify your tsconfig.json as following:

```
...
"typeRoots": [
      "./node_modules/types-redux-orm",
      "./node_modules/@types"
    ],
...
```

## Recipes

### Model

```javascript
import { attr, IORMId, IResourceCommonState, ITableState, Model, ORM } from 'redux-orm'

export class Test extends Model<ITestStateItem, IFetchIndicatorState> {
  static modelName = 'Test'

  static fields = {
    test: attr(),
    isFetching: attr({ getDefault: () => false }),
    id: attr()
  }
}

// core data which we do not have defaults for
export interface ITestStateItem {
  test: string
}

// optional data we provide defaults for
export interface IFetchIndicatorState {
  isFetching: boolean
}

// id attr is added automatically by redux-orm therefore we have IORMId interface
export type ITestState = ITableState<ITestStateItem & IORMId & IFetchIndicatorState>

export interface ITestORMState extends IResourceCommonState {
  Test: ITestState
}

interface ITestORMModels {
  Test: typeof Test
}

const orm = new ORM<ITestORMState>()
orm.register<ITestORMModels>(Test)
export default orm
```

### Reducer

```javascript
interface ITestDTO {
  test: string
}

const reducerAddItem = (state: ITestORMState, action: ActionMeta<ITestDTO, any>): ITestORMState => {
  const session = orm.session(state)
  session.Test.upsert<ITestStateItem>(action.payload)
  return session.state
}
```

### Selector

```javascript
import { createSelector as createSelectorORM, IORMId, ISession } from 'redux-orm'

interface ITestDisplayItem {
  test: string
}
type ITestDisplayItemList = ITestDisplayItem[]

export const makeGetTestDisplayList = () => {
  const ormSelector = createSelectorORM<ITestORMState>(orm, (session: ISession<ITestORMState>) =>
    session.Test
      .all<typeof Test, ITestStateItem, IFetchIndicatorState>()
      .toRefArray()
      .map((item) => ({ ...item }))
  })
  return createSelector<IRootState, ITestORMState, ITestDisplayItemList>(
    ({ test }) => test,
    ormSelector
  )
}
```
