# structure

+-- src 
|  +-- store        // ngrx
|  |  +-- root.actions.ts 
|  |  +-- root.selectors.ts 
|  |  +-- root.effects.ts 
|  |  +-- root.state.ts 
|  |  +-- root.reducer.ts 
|  |  +-- index.ts 
|  +-- core 		// imported ONLY in the app module
                    // les services qui sont singleton et réutilisés dans plus d'un module de fonctionnalités
|  +-- entities 
|  |  +-- user 
|  |  +-- user-role
|  +-- shared       // les composants, directives et tuyaux qui sont partagés et utilisés dans plusieurs modules fonctionnels
                    // Ce module sera importé dans tous les modules fonctionnels qui nécessitent les composants partagés
|  |  +-- components 
|  |  +-- directives 
|  |  +-- pipes 
|  |  +-- shared.module.ts 
|  |  +-- index.ts 
|  +-- utils 
|  +-- models       // seulement si les interfaces sont utilisées dans plusieurs fonctionnalités et/ou modules.
|  +-- features 
|  |  +-- users 
|  |  +-- roles
|  +-- app.component.ts
|  +-- app.component.html
|  +-- app.module.ts


# Simplifier les déclarations d'importation

## habituellement : 

```ts
import { Employee } "../../../../models/employee.ts"
```

## astuce 1

- on ajoute un fichier : index.ts

```
/models/Model1.ts
/models/Model2.ts
/models/Model3.ts
/models/index.tx
```

index.ts

```ts
export * from './employees.ts';
```

- utilisation :

```ts
   import { Model2 } from '../../../../models';
// au lieu de :
// import { Model2 } "../../../../models/model2.ts"
```

## astuce 2 

tsconfig.json

```json
{
     "compilerOptions": {
     "baseUrl": "./",
     "paths": {
       "@models/*": ["src/app/models/*"],
       "@models": ["src/app/models/index"]       
     }
   }
 }
```

- utilisation :

```ts
 import { Employee } '@models';
```

## exemple général

tsconfig.json

```json
{
   "paths": {
     "@models/*": ["src/app/models/*"],
     "@models": ["src/app/models/index"],
     "@core/*": ["src/app/core/*"],
     "@core": ["src/app/core/index"],
     "@shared/*": ["src/app/shared/*"],
     "@shared": ["src/app/shared/index"],
     "@utils/*": ["src/app/utils/*"],
     "@utils": ["src/app/utils/index"],
     "@data/*": ["src/app/entities/*"],
     "@data": ["src/app/entities/index"],
   }
 }
```

# clean architecture

- L'idée principale de l'architecture propre est de conserver la logique métier de base et la logique applicative en tant qu'entités distinctes, de sorte que les modifications apportées à l'une n'affectent pas nécessairement l'autre. 
- En utilisant cette architecture, nous rendons le domaine ou les entités complètement indépendants de la couche de présentation, de sorte qu'aucun changement dans l'interface utilisateur ne peut affecter le domaine ou une entité. 
- En utilisant cette architecture comme base, nous avons la liberté d'avoir un système suffisamment flexible pour faire face aux changements génériques, aux changements d'interface utilisateur, aux changements d'interface, et même aux changements de technologie, où l'application centrale reste la même et est complètement indépendante des couches de présentation, des bases de données et de l'infrastructure.
- L'architecture propre est un moyen d'isoler une application des frameworks, de l'interface utilisateur et des bases de données et de s'assurer que les composants individuels sont testables. 
- Elle s'appuie sur les principes SOLID

```
src/
├─ domain/
├─ data/
├─ presentation/
```

- Domaine :
  - Ce dossier peut contenir tous les modèles d'entreprise, les cas d'utilisation, les interacteurs et les abstractions de référentiel pour l'application. 
  - Il s'agit d'une logique commerciale qui ne sera modifiée QUE si les exigences commerciales changent.
- Data :
  - Ce dossier peut contenir tous les processeurs de l'application, les implémentations du référentiel, les modèles de sources de données et les mappeurs.
- Presentation :
  - Ce dossier peut contenir toute l'interface utilisateur de l'application. 
  - Tous les fichiers de style et les composants qui s'assemblent pour former 


## Entity

- Les couches de l'application ont une hiérarchie. Les entités sont au sommet, et l'interface utilisateur est au bas de l'échelle. 
- Une couche ne doit pas avoir de dépendance vis-à-vis d'une autre couche sous-jacente. 
- Par exemple, l'entité ne doit rien savoir de l'interface utilisateur. Aussi trivial que cela puisse paraître, l'entité est probablement la partie la plus cruciale d'une architecture propre. 
- C'est là que je commence à concevoir des fonctionnalités totalement nouvelles. 
- C'est la partie que je protège le plus des changements. 
- Bien qu'elle ne figure pas sur le diagramme, l'Entité circule entre toutes ces couches.

```ts
export interface Todo {
  uuid: string
  title: string
  description?: string
  status: 'pending' | 'done'
}
```

- Cela semble assez simple, non ? Oui, une entité peut être aussi simple qu'une interface Typescript. 
- L'idée de base est d'inclure uniquement les propriétés qui décrivent le domaine d'une nouvelle fonctionnalité. 
- Tout état qui peut être dérivé de ces propriétés n'a pas sa place ici.
- L'une des erreurs typiques consiste à ajouter à l'entité des informations supplémentaires qui facilitent le rendu. 
- Chaque fois que vous modifiez l'entité, vous devez revérifier que les nouvelles données appartiennent au domaine. 
- Ces informations doivent être pertinentes, quelle que soit l'interface utilisateur, le cadre de gestion des données ou l'API.


## data layer

- Le rôle de cette couche est de fournir une chaîne d'outils pour l'entité. 
- De quelles opérations avez-vous besoin ? Quelles sont les conditions limites avant/après l'opération ? Combien de fois l'adaptateur (API) est-il appelé ? Avez-vous besoin de mises à jour optimistes ? Qu'en est-il du tri, du filtrage et de la pagination ? Peut-être avez-vous également besoin d'effectuer des recherches ? Et vous avez probablement besoin d'opérations spécialisées, telles que "done/undone" pour un élément "to-do".

- Classe avec état interne. Elle peut utiliser des sujets/observables RxJs.
- Toute bibliothèque inspirée de Redux. Dans ce cas, Facade déclenchera des actions au lieu d'appeler directement les méthodes de la couche de données.
- Toute autre bibliothèque de gestion d'état.
- Facade peut appeler directement l'adaptateur. Essentiellement, il ignore la couche de données si vous n'avez pas besoin d'une logique de mise en cache.

```ts
@Injectable({
  providedIn: 'root',
})
export class TodosDataService {
  constructor(
    private todosAdapter: TodosAdapterService,
    private todosRepo: TodosRepository,
  ) {}

  getTodos(): Observable<Todo[]> {
    return this.todosAdapter
      .getTodos()
      .pipe(tap((todos) => this.todosRepo.setTodos(todos)))
  }
}
```

## Adapter

- À proprement parler, l'adaptateur appartient également à la couche de données. 
- C'est un concept puissant qui permet de s'assurer que l'application est bien isolée de l'API et de ses changements potentiels. 
- Les services de données dépendent de l'abstraction de l'adaptateur que nous contrôlons entièrement. 
- Il s'agit d'une mise en œuvre du principe d'inversion des dépendances : je crée une classe abstraite pour l'adaptateur, puis je l'utilise dans les services de données. 
- J'écris également une implémentation de l'adaptateur qui est entièrement cachée du reste de l'application. 
- Par conséquent, la couche de données dicte ses exigences techniques pour les implémentations de l'adaptateur. 
- Même si les données circulent de l'implémentation de l'adaptateur vers les services de données, l'adaptateur dépend toujours de la couche de données, et non l'inverse.

```ts
import { Observable } from 'rxjs'
import { Todo } from './types'

export type CreateTodoDto = Omit<Todo, 'uuid' | 'status'>

export class TodoNotFoundError extends Error {}

export abstract class TodosAdapterService {
  abstract getTodos(): Observable<Todo[]>

  abstract getTodoByUuid(uuid: string): Observable<Todo | null>

  abstract createTodo(todo: CreateTodoDto): Observable<Todo>

  abstract deleteTodo(uuid: string): Observable<void>

  /**
   * @throws TodoNotFoundError
   */
  abstract updateTodo(todo: Todo): Observable<Todo>

  /**
   * @throws TodoNotFoundError
   */
  abstract updateTodoStatus(
    uuid: string,
    status: Todo['status']
  ): Observable<Todo>
}
```

## Facade

- Dans l'exemple d'aujourd'hui, une façade est un objet qui sert d'interface entre l'interface utilisateur et la couche de données. 
- Chaque fois que l'interface utilisateur a besoin de charger des tâches ou d'en créer une nouvelle, elle appelle l'une des méthodes de la façade et reçoit un résultat sous forme d'observable.
- La façade, quant à elle, peut être n'importe quoi à l'intérieur.
- Dans des scénarios simples, j'appelle directement les méthodes des adaptateurs si je n'ai pas besoin de mise en cache ou de gestion des données.
- Dans d'autres cas, je peux déclencher une action de type redux, par exemple dispatch(loadTodos()), puis écouter les actions loadTodosSuccess et loadTodosFailure qui suivent.
- Je peux également transmettre l'appel de la façade à un autre service qui orchestre l'interaction avec les adaptateurs. Il peut s'agir d'un service auto-écrit basé sur RxJS Subjects ou d'un service tiers comme ceux de @ngrx/data (à ne pas confondre avec le NgRx nu) !

```ts
import { todos$ } from './todos.repository'
import { TodosDataService } from './todos-data.service'

export class TodosFacadeService {
  todos$: Observable<Todo[]> = todos$

  constructor(
    private todosData: TodosDataService
  ) {}

  getTodos(): Observable<Todo[]> {
    const todos$ = this.todosData.getTodos()

    todos$.subscribe()

    return todos$
  }
}
```

## conclusion

- J'ai distribué la responsabilité sur différentes classes. 
- Le service de données est censé demander des données à l'adaptateur, sauvegarder les données dans le référentiel et orchestrer des mises à jour optimales si nécessaire. 
- Le service de données définit comment modifier l'état après chaque opération.
- La façade, quant à elle, expose l'API de données à l'interface utilisateur. 
- Elle peut demander la liste des tâches ou en créer une nouvelle, puis recevoir la réponse de l'observable unifié todos$ qui masque toute la complexité des réponses. 
- En même temps, vous pouvez remarquer que j'utilise subscribe() dans la méthode de la façade et que je renvoie ensuite un observable. J'ai pris cette décision pour faciliter la logique de l'application. 
- Parfois, les composants qui déclenchent une opération et ceux qui reçoivent le résultat sont différents. Ils ont également des cycles de vie différents. 
- Dans cette application to-do, il arrive qu'un composant déclencheur soit détruit juste après avoir demandé des données. Je dois donc m'assurer que quelque chose d'autre recevra le résultat et gardera au moins un abonnement actif. 
- Facade comble commodément cette lacune en introduisant un subscribe() obligatoire à l'intérieur. En outre, il garantit que le service de données sous-jacent n'a pas de logique supplémentaire qui n'est pertinente que pour les consommateurs de données.

## UI

- Pourquoi, l'interface utilisateur a aussi une logique ! C'est une logique différente cependant. 
- L'interface utilisateur parle exclusivement à la façade. 
- Le travail de l'IU est d'appeler la façade au bon moment, par exemple lors de l'initialisation d'un composant ou d'une action spécifique de l'utilisateur. 
- En outre, l'interface utilisateur est responsable de la gestion de son état. Tout l'état ne va pas à la couche de données. La couche UI doit gérer l'état spécifique à l'UI.
- Il existe de nombreuses approches pour gérer l'état de l'interface utilisateur. Et là encore, le choix dépend des exigences de l'entreprise. Parfois, il est acceptable de stocker l'état simplement dans un composant. Dans d'autres cas, il doit y avoir un moyen d'échanger des données entre les composants de l'interface utilisateur.


- clean-architecture-angular2

```
                                                                              -------- ADAPTER -----------------------
                                                                              ABSTRACT 
UI ---------------------------------- FACADE ----------------------------------------- DATA ------------------------
                              * expose l'API de données
                              * déclencher une action de type redux, 
                              par exemple dispatch(loadTodos()), 
                              puis écouter les actions loadTodosSuccess 
                              et loadTodosFailure qui suivent

                              TodosFacadeService                              TodosDataService              TodosRepository
todosFacade.activeTodos$        activeTodos$                                    
todosFacade.updateTodoStatus    todosData.updateTodoStatus                      updateTodoStatus              todosRepo.queryTodo
                                                                                                              todosRepo.updateTodo

...................................

todos-main
  <router-outlet></router-outlet>
  this.todosFacade.getTodos()

TodosDataModule
  provide: TodosAdapterService,
  useClass: LocalTodosAdapterService,
```

- clean-architecture-angular

```
/base
    abstract mapper
        abstract mapFrom
        abstract mapTo
    interface UseCase
        execute

/data
  DataModule
      providers: [
          userLoginUseCaseProvider,
          userRegisterUseCaseProvider,
          getUserProfileUseCaseProvider,
          { provide: UserRepository, useClass: UserImplementationRepository },
  /repositories
      UserImplementationRepository extends UserRepository
          login {...}
          register {...}
          getUserProfile {...}
      /entities
          user-entity
      /mappers
          user-repositories.mapper extends Mapper
              mapFrom {...}
              mapTo {...}
/domain
    /repositories
        abstract UserRepository
            login 
            register
            getUserProfile
  /models
      user.model
  /usecases
      get-user-profile.usecase implements UseCase
        execute(): Observable<UserModel> {
            return this.userRepository.getUserProfile();
        }     
      user-login.usecase implements UseCase
        execute(params: { username: string, password: string },): Observable<UserModel> {
            return this.userRepository.login(params);
        }    
      user-register.usecase implements UseCase
        execute(params: { phoneNum: string; password: string },): Observable<UserModel> {
            return this.userRepository.register(params);
        }    

/presentation
    constructor(private GetUserProfile: GetUserProfileUseCase) { 
      this.GetUserProfile.execute(...
    }  
```

- angular-toh-hexagonal

```
/domain
    /ports
        i-display-hero-detail
            hero: Hero | undefined
            askHeroDetail(id: number): Observable<void>
            askHeroNameChange(newHeroName: string): Observable<void>    
        i-display-heroes        // (1)
            heroes: Hero[]
            filter: string
            
            askHeroesList(): Observable<void>
            askHeroesFiltered(filter: string, allowEmpty?: boolean): Observable<void>
            askHeroCreation(heroName: string): Observable<void>
            askHeroDeletion(hero: Hero): Observable<void>
        i-display-messages
            ...
        //
        i-manage-heroes         // (2)
            getHeroes(): Observable<Hero[]>
            searchHeroes(term: string): Observable<Hero[]>
            getHero(id: number): Observable<Hero> 
            addHero(hero: Hero): Observable<Hero>
            updateHero(hero: Hero): Observable<Hero>
            deleteHero(id: number): Observable<number>    
        i-manage-messages
            ...
  
    /models
        hero
    
    HeroDetailDisplayer implements IDisplayHeroDetail
    HeroesDisplayer implements IDisplayHeroes         // (1).2
        heroes: Hero[] = [];
        filter: string = '';
        //
        constructor(@Inject('IManageHeroes') private _heroesManager: IManageHeroes, ...)
        //
        askHeroesList(): Observable<void> {
            this.heroes = _heroesManager.getHeroes()          // (2).1 adapter           
        }
    MessagesDisplayer implements IDisplayMessages

/adapters
    hero-adapter.service implements IManageHeroes {
        getHeroes(): Observable<Hero[]> {                       // (2).2
          return ....http.....
        }
    in-memory-data.service implements InMemoryDbService {...}
    message-adapter.service  implements IManageMessages {...}

/components
    AppModule
        providers: [
            // Inject domain classes into components
            // pour la vue
            {provide: 'IDisplayHeroDetail', useClass: HeroDetailDisplayer},
            {provide: 'IDisplayHeroes', useClass: HeroesDisplayer},             // (1) port
            {provide: 'IDisplayHeroesSearch', useClass: HeroesDisplayer},
            {provide: 'IDisplayMessages', useClass: MessagesDisplayer},
            //
            // Inject adapters int domain classes
            // pour des services externe
            {provide: 'IManageHeroes', useClass: HeroAdapterService},           // (2) port
            {provide: 'IManageMessages', useClass: MessageAdapterService}

    /heroes
        heroes.component
            constructor(@Inject('IDisplayHeroes') public heroesDisplayer: IDisplayHeroes)        
            heroesDisplayer.askHeroesList()       // (1).1 domain


                            PORT                           PORT
VUE ---------------------> DOMAIN ----------------------> ADAPTER

- des objets appelés ports sont utilisés, et ils sont mis en œuvre par des classes d'interface. Cela affaiblit le couplage entre les éléments de notre architecture
- Dans une architecture hexagonale, l'ensemble du code lié au domaine est isolé
- Une bonne pratique dans l'architecture hexagonale est de garder le code lié au domaine indépendant de tout framework, afin de s'assurer qu'il est fonctionnel pour tout type d'adaptateur. Mais dans notre code, le domaine est fortement dépendant des objets Angular et rxjs, on ne peut pas faire autrement sans bidouiller
- remarques :
    - pour la meme classe : "HeroesDisplayer" on a 2 jetons differents
        AppModule
          {provide: 'IDisplayHeroes', useClass: HeroesDisplayer},
          {provide: 'IDisplayHeroesSearch', useClass: HeroesDisplayer},

    - le 1er jeton : 'IDisplayHeroes' est utilisé par : DashboardComponent et HeroesComponent 
    - le 2eme jeton : 'IDisplayHeroesSearch' est utilisé par : HeroSearchComponent 
    - but :
        - le 1er jeton (1ere instance) dispose d'une liste de heros qui peut etre partagé par : DashboardComponent et HeroesComponent
        - le 2eme jeton (2eme instance) dispose de sa propre liste de heros
- L'essence principale de l'architecture hexagonale consiste à disposer d'adaptateurs interchangeables permettant à notre application d'être pilotée indifféremment par un humain, un système ou des tests. 
```


```ts

```


```ts

```


```ts

```


```ts

```


```ts

```


```ts

```


```ts

```


```ts

```


```ts

```


```ts

```


```ts

```


```ts

```


