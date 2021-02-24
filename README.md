# Capitulo 2: Un enfoque mas funcional de Go

En entornos de microservicios, la mayoría de nuestro código es privado al proyecto, o mas bien propio del microservicio, nuestro código es generalmente interno y privado, por los que la implementación interna del código, no tiene porque ser compleja ni escalable ni adaptable. 

Esa es la gracia de los microservicios, son soluciones adaptadas a un problema puntual, donde las interfaces del microservicio (REST, GRPC, etc) es todo lo que se usa desde afuera.

### Programamos Go como si fuera Java...

Cuando en Go definimos una estructura, asociamos código a una estructura y generalizamos el uso de esa estructura con una interfaz (como lo hice en el ejemplo anterior), básicamente estamos programando en forma orientada a objetos.

Se suele decir que Go no es orientado a objetos, pero sin embargo en la definición misma del lenguaje estos artefactos de Go, explícitamente hacen referencia a la programación OO. 

Podemos citar muchas referencias a ésto desde el mismo sitio de [Go](https://golang.org/doc/effective_go#interfaces_and_types), y la mayoría de los libros que he leído expresan conceptos de la misma forma.

Ahora bien, si en lugar de tomar un enfoque OO, aprovechamos las capacidades de Go para programar en forma funcional, podemos encontrarnos con un código mas prolijo y directo.

### Go en forma Funcional ? se puede.

Siguiendo los lineamientos de single Responsability debería ser bastante común en nuestro código tener servicios con una sola función.

Analicemos que significa en el código anterior, la siguiente estructura :

```
// HelloService es el servicio de negocio
type HelloService struct {
	dao IHelloDao
}
```

Es básicamente encapsular un puntero a una estructura que define una función. Ademas este tipo de estructuras es un antipatrón de programación OO, muy popularizado en Java con EJB, cuando no se sabia como separar capas.

No estoy en contra de las estructuras, pero este tipo de estructuras es solo un pasamanos a una llamada a una o varias funciones, no tiene razón de existir porque no tiene estado real en nuestro sistema. 

Que tal si nos evitamos todo esto y vamos directo a una solución donde no existan estructuras innecesarias ?

Nuestro archivo main, entonces no necesita crear ningún instancia, solo llamamos a una función :

```
func main() {
	fmt.Println(service.SayHello())
}
```

Nuestro DAO es muy simple también, es solo una función.

```
func Hello() string {
	return "Holiiiiis"
}
```

Nuestro Service es un poquito mas complejo, pero no llega a ser tan complejo como usar interfaces :

```
// Nos va a permitir mockear respuestas para los tests
var sayHelloMock func() string = nil

// SayHello es nuestro negocio
func SayHello() string {
	if sayHelloMock != nil {
		return sayHelloMock()
	}

	return dao.Hello()
}
```

Dado que nuestro servicio es algo que debemos poder testear usando mocks de DAO, no nos quedan muchas opciones que permitirnos mockear esta función con un puntero a función. 
En caso que el if nos cause ruido, podemos apuntar directamente el puntero sayHelloMock a la función original, y nos evitamos ese if de mas.

El test en cuestión es el siguiente :

```
// Cuando testeamos la reescribimos con el
// mock que queramos
sayHelloMock = func() string {
	return "Hello"
}

assert.Equal(t, "Hello", SayHello())
sayHelloMock = nil
```

La estrategia de utilizar un puntero a una función, conceptualmente es la misma que utilizar una interfaz, en su forma mas simple un puntero a una función define una interfaz a respetar.

Es medio hacky ? si pero como dije antes, si algo puede ser hacky es el test.

### Ventajas de este enfoque 

Este concepto de programar en forma funcional simplifica mucho la programación tradicional, y en la mayoría de las soluciones es el balance ideal porque es la forma mas simple de definir, entender y mantener código. 

- No tenemos que escribir una interfaz, ni una estructura, tan solo para mockear
- No hacemos DI
- Single Responsibility
- Código mas simple de leer y mantener
- Podemos visualizar mejor la programación declarativa del paradigma funcional
- Llevamos el concepto de [Interface segregation](https://en.wikipedia.org/wiki/Interface_segregation_principle) a su mínima expresión, una función, algo deseable en POO

Incluso podemos hacer strategy si planificamos bien los factories, sin necesidad del uso de interfaces.

## Opinión personal sobre POO

La programación OO esta muy bien, solo que esta muy subestimada la forma en la que se debe realizar. Programar OO es muy complejo, y cuando los proyectos crecen, no se realiza el mantenimiento adecuado, por lo que en general terminamos teniendo código espagueti.

El libro Domain Driven Design de Erik Evans, expresa una forma conceptualmente adecuada de implementar POO, sin embargo es muy raro ver algo claro y bien implementado.

Muchos desarrolladores entienden que el concepto de Clean Architecture y DDD solo va en generar interfaces hacia todo lo que entra y sale del negocio, pero olvidan lo fundamental, respetar los patrones básicos para que el código sea sustentable, que es donde se encuentra el balance de simplicidad necesario.

La programación OO no es aplicar la misma regla y norma a todo, sino, requiere el uso del energía cerebral para que nuestros diseños tengan la complejidad justa y necesaria. Y eso es muy difícil de adquirir. 

Y a su vez, la POO requiere un mantenimiento adecuado, el refactor continuo debe ser una norma, y no siempre los equipos de desarrolladores lo entienden asi.

Por otro lado un enfoque funcional es mucho mas simple, el refactor es simple, lo que debemos adoptar es simplemente una separación clara del negocio con las dependencias que usa y que usan al mismo. Teniendo esa separación en capas bien lograda, el resultado es elegante, simple y sobre todo, muy eficiente.

En las empresas en general mas de la mitad de los desarrolladores tendrán poca experiencia, muchos de ellos estarán dando sus primeros pasos, por lo que ésta simplicidad es bienvenida.

## Nota

Esta es una serie de tutoriales sobre patrones simples de programación en GO.

[Tabla de Contenidos](https://github.com/nmarsollier/go_index)
