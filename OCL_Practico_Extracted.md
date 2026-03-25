# MÉTODOS FORMALES — Práctico de OCL
## Full Text Extraction from p2_OCL.pdf

---

## EJERCICIO 1: Dado el diagrama de clases siguiente:

### Diagrama de Clases — Ejercicio 1

The diagram contains the following classes (inferred from OCL expressions throughout the exercise):

**Class: `Person`**
- Attributes: `firstName: String`, `lastName: String`, `age: Integer`, `gender: Gender`, `birthDate: Date`, `isMarried: Boolean`
- Operations: `income(d: Date): Integer`
- Associations:
  - `employee` → `Company` (works at)
  - `managedCompanies` → `Company` (manages)
  - `Marriage [husband]` / `Marriage [wife]` → association class `Marriage`
  - `Job` → association class `Job`
  - `BankAccount` → via `customer` role

**Class: `Company`**
- Attributes: `numberOfEmployees: Integer`, `name: String`
- Association: `employee` → `Person` (set of employees)

**Class: `BankAccount`**
- Associations: `customer` → `Person`

**Association Class: `Marriage`**
- Roles: `husband: Person`, `wife: Person`

**Association Class: `Job`**
- Attributes: `salary: Integer`
- Roles: `employer: Company`, (employee: Person)

**Enum: `Gender`**
- Values: `Gender::female`, `Gender::male`

---

### Parte 1 — Expresar en lenguaje natural (traducir OCL a español):

**(a)** En el contexto de `Person`:
```ocl
isMarried implies age > 15
```
> **Lenguaje natural:** Si una persona está casada, entonces su edad es mayor que 15.

---

**(b)**
```ocl
context Company
inv: numberOfEmployees = employee->size()
```
> **Lenguaje natural:** En el contexto de `Company`, invariante: el atributo `numberOfEmployees` es igual al número de empleados (tamaño de la colección `employee`).

---

**(c)**
```ocl
context Person::income(d: Date) : Integer
pre:  d.laterThan(self.birthDate)
post: if age < 18
      then result < 100
      else result < 200
      endif
```
> **Lenguaje natural:** En el contexto de la operación `income(d: Date)` de la clase `Person`:
> - **Precondición:** la fecha `d` es posterior a la fecha de nacimiento de la persona.
> - **Postcondición:** si la persona tiene menos de 18 años, el resultado es menor que 100; en caso contrario, el resultado es menor que 200.

---

**(d)**
```ocl
BankAccount.allInstances().customer
  ->collect(p: Person | p.managedCompanies)
  ->asSet()
  ->size() = 3
```
> **Lenguaje natural:** El conjunto (sin duplicados) de todas las empresas administradas por los clientes de cualquier cuenta bancaria tiene exactamente 3 elementos. Es decir, entre todos los clientes de todas las cuentas bancarias, el total de empresas distintas que administran es exactamente 3.

---

**(e)** En el contexto del diagrama dado, ¿hay diferencia en los valores que devuelve la expresión **(d)** y los que devuelve la siguiente expresión? Justifique la respuesta.

```ocl
BankAccount.allInstances().customer
  ->collect(p: Person | p.managedCompanies)
  ->size() = 3
```
> **Pregunta:** ¿Difieren los resultados de la expresión **(d)** y esta nueva expresión?
>
> **Respuesta esperada:** Sí, pueden diferir. La expresión **(d)** usa `->asSet()` antes de `->size()`, lo que elimina duplicados: cuenta el número de empresas *distintas* administradas. La nueva expresión **(e)** omite `->asSet()`, por lo que `->size()` cuenta el total de *apariciones* de empresas en la colección (con repeticiones). Si una misma empresa es administrada por más de un cliente, se contará varias veces en **(e)** pero solo una vez en **(d)**.

---

**(f)** En el contexto de `Person`, explique la diferencia entre la expresión `Marriage[husband]` y la expresión `Marriage[wife]`.

> **Respuesta esperada:** 
> - `Marriage[husband]` navega desde una instancia de `Person` hacia los objetos de la clase asociación `Marriage` en los que esa persona juega el rol de **esposo** (`husband`). Devuelve el/los matrimonios donde la persona es el marido.
> - `Marriage[wife]` navega hacia los objetos `Marriage` en los que la persona juega el rol de **esposa** (`wife`). Devuelve el/los matrimonios donde la persona es la mujer.
> Ambas expresiones devuelven una colección de instancias de la clase asociación `Marriage`, pero filtradas por el rol que ocupa `self` en esa asociación.

---

**(g)**
```ocl
context Person
inv:
  self.Marriage[husband]->size() = 1 implies self.gender = Gender::female
  and
  self.Marriage[wife]->size() = 1 implies self.gender = Gender::male
```
> **Lenguaje natural:** En el contexto de `Person`, invariante: si una persona tiene exactamente un matrimonio en el que es esposo (`husband`), entonces su género es femenino (`female`); y si tiene exactamente un matrimonio en el que es esposa (`wife`), entonces su género es masculino (`male`).
>
> *(Nota: tal como está escrito esto es un invariante contradictorio/incorrecto respecto a la semántica esperada — probablemente el ejercicio pide detectar ese error o la intención era la inversa.)*

---

**(h)** En el contexto de `Person`:
```ocl
self.Job->collect(job |
  Tuple{ salario: Integer = job.salary,
         compañia: Company = job.employer }
)
```
> **Lenguaje natural:** Para cada `Person`, se recorre la colección de sus trabajos (`Job`) y se construye una colección de tuplas, donde cada tupla contiene:
> - `salario`: el salario del trabajo (`job.salary`)
> - `compañia`: la empresa empleadora (`job.employer`)

---

### Parte 2 — Expresar en OCL:

**(a)** La longitud del nombre (`firstName`) de una persona siempre es inferior a 20 caracteres; lo mismo vale para la longitud de su apellido (`lastName`).

```ocl
context Person
inv: self.firstName.size() < 20 and self.lastName.size() < 20
```

---

**(b)** Cualquiera que administre una empresa es un empleado de esa empresa. Escribir primero el invariante considerando el contexto `Person` y luego considerando el contexto `Company`.

```ocl
-- Contexto Person:
context Person
inv: self.managedCompanies->forAll(c | self.employer->includes(c))

-- Contexto Company:
context Company
inv: self.managedBy->forAll(p | self.employee->includes(p))
```

---

**(c)** Toda empresa tiene al menos un empleado de género femenino (`female`).

```ocl
context Company
inv: self.employee->exists(p | p.gender = Gender::female)
```

---

**(d)** Ninguna persona puede tener más de 5 cuentas bancarias.

```ocl
context Person
inv: self.BankAccount->size() <= 5
```

---

**(e)** Ningún cliente del banco puede tener más de 5 cuentas bancarias (un cliente es una persona que tiene al menos una cuenta).

```ocl
context Person
inv: self.BankAccount->size() >= 1 implies self.BankAccount->size() <= 5
```

---

**(f)** Nadie puede tener dos empleos en compañías que tienen idénticos nombres.

```ocl
context Person
inv: self.Job->collect(job | job.employer.name)->asSet()->size() = self.Job->size()
```
*(Alternativa: todos los nombres de las compañías empleadoras son distintos entre sí.)*

---

## EJERCICIO 2: Dado el siguiente diagrama de clases:

### Diagrama de Clases — Ejercicio 2

The diagram contains the following classes (inferred from OCL expressions throughout the exercise):

**Class: `Persona`**
- Attributes: `nroDoc: Integer`, `tipoDoc: TipoDocumento`, `nombre: String`, `apellido: String`, `/edad: Integer` (derived), `genero: Genero`, `domicilio` → `Circuito` (lives in)
- Associations:
  - vota en → `Mesa`
  - afiliada a → `Partido` (via `Afiliado`)
  - `Matrimonio` (cónyuge 1 / cónyuge 2)

**Subclass: `Afiliado` extends `Persona`**
- Attributes: `estudios: NivelEstudios`
- Associations: afiliado a → `Partido`; puede ser presidente de → `Partido`

**Class: `Partido`**
- Attributes: `nombre: String`, `ambito: Ambito` (nacional / provincial)
- Associations: `afiliados` → set of `Afiliado`; `presidente` → `Afiliado`; pertenece a → `Circuito`

**Class: `Mesa`**
- Attributes: `nroMesa: Integer`, `{static} máximo: Integer` (init = 100)
- Associations: `electores` → set of `Persona`; `presidente` → `Afiliado`; pertenece a → `Circuito`

**Class: `Circuito`**
- Attributes: (identifier)
- Associations: `mesas` → set of `Mesa`; `partidos` → set of `Partido`

**Association Class: `Matrimonio`**
- Attributes: `fecha: Fecha`, `fechaDivorcio: Fecha` (optional/null if still married)
- Roles: `conyuge1: Persona`, `conyuge2: Persona`

**Class: `Fecha`**
- Operations: `años(otraFecha: Fecha): Integer` (returns difference in years between two dates)

**Enum: `TipoDocumento`**: `LE`, `LC`, `DNI`, ...

**Enum: `Genero`**: `masculino`, `femenino`

**Enum: `NivelEstudios`**: `primario`, `secundario`, `universitario`, ...

---

### Parte 1 — Atributo estático inicial

**1.** Especificar en OCL el valor inicial del atributo estático `máximo` de la clase `Mesa` en 100.

```ocl
context Mesa
init máximo: 100
```

---

### Parte 2 — Atributo derivado

**2.** Especificar el valor del atributo derivado `edad` de la clase `Persona`, expresado como la cantidad de años entre la fecha de nacimiento y la fecha corriente. Nota: el método `años` de la clase `Fecha` devuelve la diferencia en años entre dos fechas.

```ocl
context Persona
derive edad: Fecha.today().años(self.fechaNacimiento)
```
*(Assuming `Fecha::today()` returns current date; `años()` returns year difference.)*

---

### Parte 3 — Cuerpos de operaciones de consulta

**(a)** Todos los afiliados de ambos géneros de un partido.

```ocl
context Partido::afiliados(): Set(Afiliado)
body: self.afiliado
```

---

**(b)** Todas las afiliadas a un partido con estudios universitarios.

```ocl
context Partido::afiliadasUniversitarias(): Set(Afiliado)
body: self.afiliado->select(a | a.genero = Genero::femenino and a.estudios = NivelEstudios::universitario)
```

---

**(c)** Los afiliados de ambos géneros a un partido de nombre dado.

```ocl
context Partido::afiliadosPorNombre(nombre: String): Set(Afiliado)
body: Partido.allInstances()->select(p | p.nombre = nombre)->collect(p | p.afiliado)->asSet()
```

---

**(d)** La cantidad de afiliados de ambos géneros de un partido.

```ocl
context Partido::cantAfiliados(): Integer
body: self.afiliado->size()
```

---

**(e)** Si una persona está afiliada a un partido dado.

```ocl
context Persona::estaAfiliada(p: Partido): Boolean
body: Afiliado.allInstances()->exists(a | a = self and a.partido = p)
```
*(or simply: `self.oclIsKindOf(Afiliado) and self.oclAsType(Afiliado).partido = p`)*

---

**(f)** Dados el nombre y apellido de una persona y el nombre de un partido, determinar si existe una persona con ese nombre y apellido que esté afiliada al partido de nombre dado.

```ocl
context Partido::existeAfiliado(nombre: String, apellido: String, nombrePartido: String): Boolean
body: Afiliado.allInstances()->exists(a |
        a.nombre = nombre and
        a.apellido = apellido and
        a.partido.nombre = nombrePartido)
```

---

**(g)** Todos los matrimonios celebrados entre dos fechas dadas.

```ocl
context Matrimonio::entreF echas(f1: Fecha, f2: Fecha): Set(Matrimonio)
body: Matrimonio.allInstances()->select(m |
        m.fecha >= f1 and m.fecha <= f2)
```

---

**(h)** Una secuencia con los matrimonios igualitarios (personas de igual sexo), ordenada por fecha de matrimonio.

```ocl
context Matrimonio::igualitariosOrdenados(): Sequence(Matrimonio)
body: Matrimonio.allInstances()
        ->select(m | m.conyuge1.genero = m.conyuge2.genero)
        ->sortedBy(m | m.fecha)
```

---

**(i)** El conjunto ordenado con los afiliados de ambos géneros de un partido que sean mayores de 60 años.

```ocl
context Partido::afiliadosMayores(): OrderedSet(Afiliado)
body: self.afiliado->select(a | a.edad > 60)->sortedBy(a | a.apellido)
```

---

**(j)** Si la cantidad de afiliados de ambos géneros a partidos nacionales en un circuito dado es mayor que la cantidad de afiliados de ambos géneros a partidos provinciales del mismo circuito.

```ocl
context Circuito::masNacionalesQueProvinciales(): Boolean
body:
  let nacionales: Integer =
    self.partido->select(p | p.ambito = Ambito::nacional)
                ->collect(p | p.afiliado)->asSet()->size() in
  let provinciales: Integer =
    self.partido->select(p | p.ambito = Ambito::provincial)
                ->collect(p | p.afiliado)->asSet()->size() in
  nacionales > provinciales
```

---

**(k)** Todos los afiliados a un partido ordenados por circuito.

```ocl
context Partido::afiliadosPorCircuito(): Sequence(Afiliado)
body: self.afiliado->sortedBy(a | a.mesa.circuito)
```
*(Assuming each `Afiliado`/`Persona` is linked to a `Mesa` which belongs to a `Circuito`.)*

---

**(l)** El nombre del/los partido/s que tiene/n el/la afiliado/a de mayor edad.

```ocl
context Partido::partidoConMayorAfiliado(): Set(String)
body:
  let maxEdad: Integer = Afiliado.allInstances()->collect(a | a.edad)->max() in
  Partido.allInstances()
    ->select(p | p.afiliado->exists(a | a.edad = maxEdad))
    ->collect(p | p.nombre)
    ->asSet()
```

---

**(m)** El par conteniendo el número de mesa y circuito en la que vota una persona de nro. y tipo de documento dados.

```ocl
context Persona::mesaYCircuito(nroDoc: Integer, tipoDoc: TipoDocumento): Tuple(nroMesa: Integer, circuito: Circuito)
body:
  let p: Persona = Persona.allInstances()
    ->any(per | per.nroDoc = nroDoc and per.tipoDoc = tipoDoc) in
  Tuple{ nroMesa: Integer = p.mesa.nroMesa,
         circuito: Circuito = p.mesa.circuito }
```

---

**(n)** Una secuencia de tuplas conteniendo los apellidos y nombres de los dos cónyuges así como la fecha del matrimonio de todos aquellos matrimonios vigentes (sin fecha de divorcio). La secuencia debe encontrarse ordenada por fecha de matrimonio y el nombre y apellido de c/u de los cónyuges debe ser, a su vez, una tupla con dos campos.

```ocl
context Matrimonio::vigentesOrdenados(): Sequence(Tuple(...))
body:
  Matrimonio.allInstances()
    ->select(m | m.fechaDivorcio.isUndefined())
    ->sortedBy(m | m.fecha)
    ->collect(m |
        Tuple{
          conyuge1: Tuple{ nombre: String = m.conyuge1.nombre,
                           apellido: String = m.conyuge1.apellido },
          conyuge2: Tuple{ nombre: String = m.conyuge2.nombre,
                           apellido: String = m.conyuge2.apellido },
          fecha: Fecha = m.fecha
        })
```

---

### Parte 4 — Invariantes

**(a)** Si el tipo de documento de una persona es `LE` el género debe ser masculino, y si es `LC`, el género debe ser femenino.

```ocl
context Persona
inv: (self.tipoDoc = TipoDocumento::LE implies self.genero = Genero::masculino)
     and
     (self.tipoDoc = TipoDocumento::LC implies self.genero = Genero::femenino)
```

---

**(b)** Todos los matrimonios anteriores al 15/07/10 deben ser entre personas de distinto género. Nota: asuma la existencia de un constructor de objetos de clase `Fecha` que recibe tres parámetros enteros para el día, mes y año.

```ocl
context Matrimonio
inv: self.fecha < Fecha.new(15, 7, 2010)
     implies self.conyuge1.genero <> self.conyuge2.genero
```

---

**(c)** Los presidentes de mesa deben contar por lo menos con estudios secundarios.

```ocl
context Mesa
inv: self.presidente.estudios >= NivelEstudios::secundario
```
*(Assuming `NivelEstudios` has an ordering: primario < secundario < universitario)*

---

**(d)** Los presidentes de mesa no deben estar afiliados a ningún partido político.

```ocl
context Mesa
inv: Partido.allInstances()->forAll(p | not p.afiliado->includes(self.presidente))
```
*(Alternatively: `self.presidente.partido->isEmpty()` if there is a direct link)*

---

**(e)** Todas las personas que votan en alguna mesa tienen una edad mayor o igual a 16.

```ocl
context Mesa
inv: self.electores->forAll(p | p.edad >= 16)
```

---

**(f)** La cantidad de electores de cada mesa de un circuito no debe superar el cociente entre la cantidad total de electores y la cantidad total de mesas del circuito, ni debe superar el valor máximo (atributo estático `máximo`) establecido para las mesas.

```ocl
context Mesa
inv:
  let totalElectores: Integer = self.circuito.mesas->collect(m | m.electores)->asSet()->size() in
  let totalMesas: Integer = self.circuito.mesas->size() in
  self.electores->size() <= (totalElectores / totalMesas)
  and
  self.electores->size() <= Mesa::máximo
```

---

**(g)** Las personas que votan, lo hacen en el circuito en donde están domiciliadas.

```ocl
context Persona
inv: self.mesa->notEmpty() implies self.mesa.circuito = self.domicilio
```

---

**(h)** La edad de las personas (medida en años) es mayor o igual a cero y la edad de los afiliados es mayor o igual a 18. ¿Es posible expresar ese invariante para `Afiliado` o estaríamos violando el principio de sustitución de Liskov? Justificar.

```ocl
context Persona
inv: self.edad >= 0

context Afiliado
inv: self.edad >= 18
```

> **Justificación Liskov:** El principio de sustitución de Liskov (LSP) establece que los invariantes de la superclase deben ser preservados por la subclase (y los invariantes de la subclase pueden ser más restrictivos). Agregar `edad >= 18` en `Afiliado` **no viola** LSP porque es un invariante más restrictivo que el de `Persona` (`edad >= 0`), es decir, todo `Afiliado` también cumple el invariante de `Persona`. Sin embargo, si se exigiera que los afiliados tuvieran `edad < 0` (relajando el invariante), sí se violaría LSP.

---

### Parte 5 — Diseño por Contrato (pre/post condiciones)

**(a)** La operación `afiliar(p: Persona)` de la clase `Partido`. Una persona se puede afiliar a un partido siempre que ya no se encuentre afiliada a algún partido y su edad sea mayor o igual a 18.

```ocl
context Partido::afiliar(p: Persona)
pre:  Partido.allInstances()->forAll(part | not part.afiliado->includes(p))
      and p.edad >= 18
post: self.afiliado = self.afiliado@pre->including(p)
```

> **Clase sugerida:** La operación podría moverse a `Persona`, ya que la precondición es principalmente sobre la persona (edad, no afiliada). Tenerla en `Persona` haría la interfaz más natural: `p.afiliarA(partido)`.

---

**(b)** La operación `desafiliar(nroDoc: Integer, tipoDoc: TipoDocumento)` de la clase `Partido`. Para poder desafiliar a alguien, la persona debe estar afiliada al partido y no debe ser su presidente.

```ocl
context Partido::desafiliar(nroDoc: Integer, tipoDoc: TipoDocumento)
pre:
  let p: Afiliado = self.afiliado->any(a | a.nroDoc = nroDoc and a.tipoDoc = tipoDoc) in
  p <> null
  and self.presidente <> p
post:
  let p: Afiliado = self.afiliado@pre->any(a | a.nroDoc = nroDoc and a.tipoDoc = tipoDoc) in
  self.afiliado = self.afiliado@pre->excluding(p)
```

> **Clase sugerida:** También podría ubicarse en `Persona`/`Afiliado` para mayor cohesión.

---

**(c)** La operación `cambiarPresidencia(nroDoc: Integer, tipoDoc: TipoDocumento)` de la clase `Partido`. Para poder realizar la operación, el nuevo presidente ya debe estar afiliado al partido.

```ocl
context Partido::cambiarPresidencia(nroDoc: Integer, tipoDoc: TipoDocumento)
pre:
  self.afiliado->exists(a | a.nroDoc = nroDoc and a.tipoDoc = tipoDoc)
post:
  let nuevo: Afiliado = self.afiliado->any(a | a.nroDoc = nroDoc and a.tipoDoc = tipoDoc) in
  self.presidente = nuevo
```

> **Clase sugerida:** Podría moverse a `Afiliado` si hay una referencia al partido al que pertenece.

---

**(d)** La operación `inscribirMatrimonio(c1, c2: Persona, fecha: Fecha)` de la clase asociación `Matrimonio`. Para poder inscribir un matrimonio ninguno de los contrayentes puede estar casado (sí pudo estarlo en el pasado pero ahora está divorciado) y la fecha del matrimonio no puede ser mayor a la fecha del día en que se realiza la inscripción.

```ocl
context Matrimonio::inscribirMatrimonio(c1: Persona, c2: Persona, fecha: Fecha)
pre:
  -- Ninguno está actualmente casado (sin fecha de divorcio = vigente)
  Matrimonio.allInstances()->forAll(m |
    not (m.fechaDivorcio.isUndefined() and
         (m.conyuge1 = c1 or m.conyuge2 = c1 or
          m.conyuge1 = c2 or m.conyuge2 = c2)))
  and fecha <= Fecha.today()
post:
  Matrimonio.allInstances() = Matrimonio.allInstances@pre()->including(
    Matrimonio{ conyuge1 = c1, conyuge2 = c2, fecha = fecha, fechaDivorcio = undefined })
```

> **Clase sugerida:** La operación podría estar en una clase `RegistroCivil` o similar, que centralice las inscripciones. En `Matrimonio` como clase asociación resulta algo forzado.

---

## SUMMARY: Classes and Associations (Both Exercises)

### Exercise 1 — Personnel/Company/Banking Model
| Class | Key Attributes | Key Associations |
|-------|---------------|-----------------|
| `Person` | firstName, lastName, age, gender, birthDate, isMarried | employee→Company, managedCompanies→Company, Marriage, Job, BankAccount |
| `Company` | numberOfEmployees, name | employee→Person |
| `BankAccount` | — | customer→Person |
| `Marriage` (assoc) | — | husband→Person, wife→Person |
| `Job` (assoc) | salary | employer→Company |
| `Gender` (enum) | male, female | — |

### Exercise 2 — Electoral/Political Model
| Class | Key Attributes | Key Associations |
|-------|---------------|-----------------|
| `Persona` | nroDoc, tipoDoc, nombre, apellido, /edad, genero | mesa→Mesa, domicilio→Circuito, Matrimonio |
| `Afiliado` (extends Persona) | estudios | partido→Partido |
| `Partido` | nombre, ambito | afiliados→Afiliado, presidente→Afiliado, circuito→Circuito |
| `Mesa` | nroMesa, {static}máximo=100 | electores→Persona, presidente→Afiliado, circuito→Circuito |
| `Circuito` | — | mesas→Mesa, partidos→Partido |
| `Matrimonio` (assoc) | fecha, fechaDivorcio | conyuge1→Persona, conyuge2→Persona |
| `Fecha` | — | años(Fecha):Integer |
| `TipoDocumento` (enum) | LE, LC, DNI | — |
| `Genero` (enum) | masculino, femenino | — |
| `NivelEstudios` (enum) | primario, secundario, universitario | — |
| `Ambito` (enum) | nacional, provincial | — |
