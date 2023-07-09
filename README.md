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
## 4. Interface Segregation Principle
## 5. Dependency Inversion Principle

