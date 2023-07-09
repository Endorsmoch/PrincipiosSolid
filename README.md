# Principios SOLID en Spring Boot
Para el Paradigma de Programación Orientada a Objetos se hace uso de unos principios como buena práctica para la generación de código, estos principios son conocidos como los Principios SOLID los cuales tienen por objetivo fomentar el desarrollo de **código limpio** el cual permite una mejor lectura y agiliza los trabajos posteriores de mantenimiento, o proyectos de escalabilidad a un producto de software previamente desarrollado.

Los cinco principios SOLID, uno por cada letra que compone al nombre de los principios, proponen las siguientes ideas:

## 1. Single Responsibility Principle
El Principio de Responsabilidad Única plantea que cado uno de los archivos que compongan a un proyecto de software tenga una responsabilidad clara y reconocible, eso quiere decir, que se espera que el contenido que conforma a dicho archivo (clase, atributos y métodos) sean referentes a la responsabilidad establecida para dicha entidad de código. 
Ejemplo de este principio dentro de Spring Boot para un microservicio, lo podemos encontrar en la siguiente clase: 

```java

    @RestController
    @RequestMapping("pago")
    @Log4j2
    public class PagoController {
        @Autowired
        private PagoService pagoService;
    
        @GetMapping
        public ResponseEntity<?> getAllPagos() {
            try {
                return ResponseEntity.ok().body(pagoService.getAllPagos());
            } catch (COAException coaException) {
                log.warn("Sin datos");
                log.error(coaException);
                return new ResponseEntity<>("Datos no encontrados", HttpStatus.OK);
            } catch (Exception exception) {
                log.error(exception);
                throw new RuntimeException(exception);
            }
        }
    
        @PostMapping
        public Pago createPago(@RequestBody Pago pago) {
            return pagoService.createPago(pago);
        }
    
        @PutMapping("/{id}")
        public ResponseEntity<?> updatePago(@RequestBody Pago pago) {
            try {
                return ResponseEntity.ok().body(pagoService.updatePago(pago));
            } catch (COAException coaException) {
                log.warn("Sin datos");
                log.error(coaException);
                return new ResponseEntity<>(coaException.getMessage(), HttpStatus.BAD_REQUEST);
            } catch (Exception exception) {
                log.error(exception);
                throw new RuntimeException(exception);
            }
        }
    
        @DeleteMapping("/{id}")
        public void deletePago(@PathVariable(value = "id") Long id) {
            pagoService.deletePago(id);
        }
    
    }
```

El ejemplo anterior nos muestra que la clase **PagoController** tiene por responsabilidad controlar las peticiones HTTP recibidas en el endpoint **"/pago"** y redirigirlas al apartado del programa que porporciona los servicios, en este caso a otra clase **PagoService** cuya responsabilidad es proporcionar los servicios REST de dicho endpoint del microservicio.

Un proyecto de Spring Boot que siga correctamente este principio, tendrá una estructura similar a la de la siguiente imagen. En dicha imagen se pueden obervar carpetas o "packages" que contienen archivos con una única responsabilidad. El uso de las carpetas sirve para identificar la responsabilidad que tienen los archivos contenidos.

<img width="200" alt="S_ProjectStrct" src="https://github.com/Endorsmoch/PrincipiosSolid/assets/78102161/37e5e254-958e-451e-828f-53cd1d8c035a">

## 2. Open/Closed Principle
Este es un principio que hace referencia a que el código debe estar creado para ser extendido y no modificado. La idea es que el código que se genere posteriormente a la creación de una entidad de código, debe estar construida considerando el código previo y no modificar lo que ya estaba escrito, ahí se observa el cerrado a modificación.

En un proyecto de Spring Boot se puede identificar este principio en la creación de las clases DTO para la presentación de datos en un endpoint. La idea es crear estas entidades que tomen lo necesario de las entidades que tienen por responsabilidad representar los objetos básicos que son necesarios dentro del microservicio.

Una de las clases entidad cuyo método *toString()* obtenido con la anotación **@Data** no es recomendable en producción:
```java
@Entity
@Table(name ="kardex")
@Data
@NoArgsConstructor
public class Kardex {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id")
    private Long id;
    @Column(name ="folio")
    private String folioKardex;
    @Column(name = "materia_id", nullable = false, precision = 30)
    private Long materia;
    @Column(name = "calificacion", nullable = false, precision = 6)
    private Double calificacion;
    @ManyToOne
    @JoinColumn( name = "alumno_id")
    private Alumno alumno;
}
```
El método *toString()* mostraría hasta el id de la entidad y eso es peligroso para la seguridad de los datos pues puede ser utilizado para futuras inyecciones SQL. En su lugar se extiende y se crea otra entidad o clase cuya responsabilidad sea cambiar la apariencia de cómo se presentan ciertos datos que son también parte de la clase **Kardex**:

```java
@Data
@NoArgsConstructor
public class KardexAlumno {
    private String folio;
    private String nombreCompleto;
    private String licenciatrua;
    private List<MateriasKardex> materiasKardex;
}
```
## 3. Liskov Substitution Principle
La Sustitución de Liskov es el principio con el que se pretende que las clases que se construyan extendiendo a una clase, superior, puedan ser utilizadas en lugar de la subclase padre y esa sustitución no debería afectar en lo absoluto al funcionamiento del sistema. Ejemplo de ello podría ser que las clases que deriven de otra, no arrojen excepciones que la clase padre no arroja.

El ejemplo anterior lo podemos observar dentro de un proyecto de Spring Boot en las entidades que extienden a **Service** haciendo uso de la anotación **@Service**. Dichas clases no arrojan nuevas excepciones, simplemente definen los métodos para la conexión con la base de datos haciendo uso de una Interfaz *Repository*:

```java
@Service
@Log4j2
public class AlumnoService {
    @Autowired
    private AlumnoRepository alumnoRepository;

    public Alumno createAlumno(Alumno alumno){
        log.info("crea alumno: "+alumno.toString());
        return alumnoRepository.save(alumno);
    }

    public Alumno updateAlumno(Alumno alumno){
        log.info("actualiza alumno: "+alumno.toString());
        return alumnoRepository.save(alumno);
    }

    public List<Alumno> getAllAlumnos(){
        return alumnoRepository.findAll();
    }

    public void deleteAlumno(Long id){
        alumnoRepository.deleteById(id);
    }
}
```
## 4. Interface Segregation Principle
Robert C. Martin dijo:
> “Make fine grained interfaces that are client specific.”

Lo anterior puede entenderse como, haz tantas interfaces sean necesarias mientras tengan a un único cliente específico. En un sistema de microservicios con Spring Boot esto puede verse reflejado en la estructura del proyecto, en donde en el paquete *Repository* se encuentran las interfaces que se comunican con el repositorio para permanencia de los datos. Existen tantas interfaces como entidades o clases **Entity**:
<img width="194" alt="I_ProjectStrct" src="https://github.com/Endorsmoch/PrincipiosSolid/assets/78102161/423f5c1e-b3b7-4e7b-b9b6-74a73a268b0f">


## 5. Dependency Inversion Principle
Este principio indica que se debe depender de las abstracciones y no de las clases concretas con las que cuenta el sistema, de esta manera reducimos el acoplamiento en el mismo. Esto puede lograrse evitando las implementaciones directas de clases en específico y haciendo uso de interfaces. Esto puede observarse en el snipet utilizado en el tercer principio. Una clase con la anotación **@Service** no hace uso de una clase que se comunica con la base de datos, sino con una interfaz.
