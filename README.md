

# Qual a melhor forma de iniciar o DataSource na nova versão do TypeORM?
# No `repositório` ou no arquivo `app-data-source.ts`?

1. Como é recomendado utilizar
## Recomendação de como utilizar com Express pelo site
[TypeORM](https://typeorm.io/example-with-express "TypeORM Express")

### Vamos criar `app-data-source.ts` onde configuramos as opções 
### iniciais de conexão com o banco de dados:

```JavaScript
import { DataSource } from "typeorm"
const myDataSource = new DataSource({
    type: "postgress",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
    entities: ["src/entity/*.js"],
    logging: true,
    synchronize: true,
})
export { myDataSource }
```

### Vamos criar uma `user.entity.ts`  entidade dentro de src/entity:
```JavaScript
import { Entity, Column, PrimaryGeneratedColumn } from "typeorm"
@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number
    @Column()
    firstName: string
}
```
#### Vamos mudar `src/app.ts`  para estabelecer a conexão com o banco de dados e 
#### começar a usar myDataSource:
```JavaScript
import * as express from "express"
import { Request, Response } from "express"
import { User } from "./entity/User"
import { myDataSource } from "./app-data-source.ts"
// estabelecendo a conexao com banco de dados
myDataSource.initialize()
    .then(() => {console.log("Data Source has been initialized!")})
    .catch((err) => {console.error("Error during Data Source initialization:", err)})
// create and setup express app
const app = express()
app.use(express.json())
// register routes
app.get("/users", async function (req: Request, res: Response) {
    const users = await myDataSource.getRepository(User).find()
    res.json(users)
})
// start express server
app.listen(3000)
```

2. Dúvida e problema encontrado

### Mesmo inicializando no Express como recomendado na documentação  
### foi necessário iniciar novamente o DataSource no repositório, 
### do contrário não funciona.

#### EXEMPLO 1 --- O arquivo `app-data-source.ts` não muda nada, sua inicialização deve ser 
#### colocada no construtor da classe de repositorio personalizada


##### O arquivo `app-data-source.ts`
```JavaScript
import { DataSource } from "typeorm"
const myDataSource = new DataSource({
    type: "postgress",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
    entities: ["src/entity/*.js"],
    logging: true,
    synchronize: true,
})
export { myDataSource }
```

##### repositorio `DoctypesRepository.ts`
```JavaScript
import { Repository } from "typeorm";
import { myDataSource } from "./app-data-source.ts"
import IDoctypesRepository from "@modules/doctypes/repositories/IDoctypesRepository";
import Doctype from "../entities/Doctype";
import ICreateDoctypesDTO from "@modules/doctypes/dtos/ICreateDoctypesDTO";

class DoctypesRepository implements IDoctypesRepository {
    private ormRepository: Repository<Doctype>;
    constructor() {
        myDataSource.initialize() // estabelecendo a conexao com banco de dados
        this.ormRepository = myDataSource.getRepository(Doctype)
    }
    async create({ type, localization, expiration, user_id }: ICreateDoctypesDTO): Promise<Doctype> {
        const doctype = this.ormRepository.create({ type, localization, expiration, user_id})
        await this.save(doctype);
        return doctype
    }
}
export { DoctypesRepository };
```
#### EXEMPLO 2  --mudar o `app-data-source.ts` incluindo a inicilização nele
#### e recomer a inicilização do contrutor do repositório

##### O arquivo `app-data-source.ts`
```JavaScript
import { DataSource } from "typeorm"
const myDataSource = new DataSource({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
    entities: ["src/entity/*.js"],
    logging: true,
    synchronize: true,
})

myDataSource.initialize()
    .then(() => {console.log("Data Source has been initialized!")})
    .catch((err) => {console.error("Error during Data Source initialization:", err)})

export { myDataSource }
```

##### repositorio `DoctypesRepository.ts`
```JavaScript
import { Repository } from "typeorm";
import { myDataSource } from "./app-data-source.ts"
import IDoctypesRepository from "@modules/doctypes/repositories/IDoctypesRepository";
import Doctype from "../entities/Doctype";
import ICreateDoctypesDTO from "@modules/doctypes/dtos/ICreateDoctypesDTO";

class DoctypesRepository implements IDoctypesRepository {
    private ormRepository: Repository<Doctype>;
    constructor() {
        this.ormRepository = myDataSource.getRepository(Doctype)
    }
    async create({ type, localization, expiration, user_id }: ICreateDoctypesDTO): Promise<Doctype> {
        const doctype = this.ormRepository.create({ type, localization, expiration, user_id})
        await this.save(doctype);
        return doctype
    }
}
export { DoctypesRepository };
```
