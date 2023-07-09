# Principios SOLID en Spring Boot
Para el Paradigma de Programación Orientada a Objetos se hace uso de unos principios como buena práctica para la generación de código, estos principios son conocidos como los Principios SOLID los cuales tienen por objetivo fomentar el desarrollo de **código limpio** el cual permite una mejor lectura y agiliza los trabajos posteriores de mantenimiento, o proyectos de escalabilidad a un producto de software previamente desarrollado.

Los cinco principios SOLID, uno por cada letra que compone al nombre de los principios, proponen las siguientes ideas:

## 1. Single Responsibility Principle
El Principio de Responsabilidad Única plantea que cado uno de los archivos que compongan a un proyecto de software tenga una responsabilidad clara y reconocible, eso quiere decir, que se espera que el contenido que conforma a dicho archivo (clase, atributos y métodos) sean referentes a la responsabilidad establecida para dicha entidad de código. 
Ejemplo de este principio dentro de Spring Boot para un microservicio, lo podemos encontrar en la siguiente clase: 
///
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
///
El ejemplo anterior nos muestra que la clase **PagoController** tiene por responsabilidad controlar las peticiones HTTP recibidas en el endpoint **"/pago"** y redirigirlas al apartado del programa que porporciona los servicios, en este caso a otra clase **PagoService** cuya responsabilidad es proporcionar los servicios REST de dicho endpoint del microservicio.
## 2. Open/Closed Principle
## 3. Liskov Substitution Principle
## 4. Interface Segregation Principle
## 5. Dependency Inversion Principle

