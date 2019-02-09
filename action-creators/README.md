Redux type safe action creators
-----

Write this:
    
    /** actions */
    export const Init = action('[App] Init')
    
    export const DataLoaded = action('[Data] Loaded')
        .map((data: Data) => ({data})
    
    export const DataLoadFailure = errorAction('[Data] Load Failure')

Instead of this
    
    /** actions */
    export enum ActionTypes {
        Init: '[App] Init',
        DataLoaded: '[Data] Loaded',
        DataLoadFailure: '[Data] Load Failed'
    } 
    
    export class Init implements Action {
        type = ActionTypes.Init
    }
    
    export class DataLoaded implements Action {
        type = ActionTypes.DataLoaded
        
        constructor(readonly data: Data) {
        }
    }
    
    export class DataLoadFailure implements ErrorAction {
        type = ActionTypes.DataLoadFailure
        
        constructor(readonly error: any) {
        }
    }
    
    
Usage example: 

    /** reducer */
    import * as actions from './actions'
    
    type RootActions = ActionType<typeof actions>
    
    export const reducer = (action: RootActions, state: State) {
        switch (action.type) {
            case Init.type: return state
            case DataLoaded.type: return state
            default: return state
        }
    }
    
    type DataActions = ActionType<[
            typeof DataLoaded,
            typeof DataLoadFailure
        ]>
    
    export const dataReducer = (action: DataActions, state: State) {
        ...
    }
    
    /** effects */
    
    // needs to be written only once in project
    function ofType<T extends RootActions["type"]>(...types: T[]): (s: Observable<RootActions>) => Observable<Extract<RootActions, { type: T }>> {
        return libOfType(...types) as any;
    }
    
    export const initEpic = actions => actions.pipe(
        ofType<Init>(ActionTypes.Init),
        switchMap(() => new DataLoaded([]))
    )
    
    export const fetchDataEpic = actions => actions.pipe(
        ofType<DataLoaded>(ActionTypes.DataLoaded),
        switchMap({data} => {...}) // type is inferred
    )
    
Additional examples
    
    // empty actions
    const EmptyAction = action('[Empty] action')
    const emptyAction = EmptyAction() // {type: '[Empty] action'}
    
    // passing parameters via object
    const FirstAction = action('[First] action')
        .withPayload({id: string})
        
    const firstAction = FirstAction({id: "1"}) // {type: '[First] action', id: '1'}
    
    
    // passing parameters via parameters
    const SecondAction = action('[Second] action')
        .map((id: string) => ({id})
    
    const secondAction = SecondAction("1") // {type: '[Second] action', id: '1'}
    
    // create factories
    function errorAction<T extends string>(type: T): ActionCreator<T, { error: any }> {
        return actionWithPayload<T, { error: any }>(type).map((error: any) => ({error})
    }
    
    const Error = errorAction("[Error] error")
    const error = Error("error") // // {type: '[Error] error', error: 'error'}
    
    // getting action type in reducer
    FirstAction.type === firstAction.type // true
    
    // action type
    type FirstActionType = ActionType<typeof FirstAction>
    
    // combining actions
    type FirstAndSecondActions = ActionType<[
        typeOf FirstAction,
        typeOf SecondAction
        ]>
    
    type FirstAndSecondActionsViaObject = ActionType<{
        FirstAction: typeOf FirstAction,
        SecondAction: typeOf SecondAction
        }>
       
    
    
Inspired by typesafe-actions