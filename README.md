Esta página introduce el concepto de estado y ciclo de vida en un componente de React.
Podemos comenzar por encapsular cómo se ve el reloj:

function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);

Convertir una función en una clase
Se puede convertir un componente de función como Clock en una clase en cinco pasos:

Crear una clase ES6 con el mismo nombre que herede de React.Component.
Agregar un único método vacío llamado render().
Mover el cuerpo de la función al método render().
Reemplazar props con this.props en el cuerpo de render().
Borrar el resto de la declaración de la función ya vacía.
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

Agregar estado local a una clase
Moveremos date de las props hacia el estado en tres pasos:

Reemplazar this.props.date con this.state.date en el método render():
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
2Añadir un constructor de clase que asigne el this.state inicial:
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

Agregar métodos de ciclo de vida a una clase
En aplicaciones con muchos componentes, es muy importante liberar recursos tomados por los componentes cuando se destruyen.

Queremos configurar un temporizador cada vez que Clock se renderice en el DOM por primera vez. Esto se llama «montaje» en React.

También queremos borrar ese temporizador cada vez que el DOM producido por Clock se elimine. Esto se llama «desmontaje» en React.

Podemos declarar métodos especiales en la clase del componente para ejecutar algún código cuando un componente se monta y desmonta:

class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
  }

  componentWillUnmount() {
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
Estos métodos son llamados «métodos de ciclo de vida».

El método componentDidMount() se ejecuta después que la salida del componente ha sido renderizada en el DOM. Este es un buen lugar para configurar un temporizador:

componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }
Nota como guardamos el ID del temporizador en this (this.timerID).

Nota como guardamos el ID del temporizador en this (this.timerID).

Si bien this.props es configurado por el mismo React y this.state tiene un significado especial, eres libre de añadir campos adicionales a la clase manualmente si necesitas almacenar algo que no participa en el flujo de datos (como el ID de un temporizador).

Eliminaremos el temporizador en el método de ciclo de vida componentWillUnmount():

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  Finalmente, implementaremos un método llamado tick() que el componente Clock ejecutará cada segundo.

Utilizará this.setState() para programar actualizaciones al estado local del componente.

class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);

Repasemos rápidamente lo que está sucediendo y el orden en que se invocan los métodos:

Cuando se pasa <Clock /> a ReactDOM.render(), React invoca al constructor del componente Clock. Ya que Clock necesita mostrar la hora actual, inicializa this.state con un objeto que incluye la hora actual. Luego actualizaremos este estado.
React invoca entonces al método render() del componente Clock. Así es como React sabe lo que se debe mostrar en pantalla. React entonces actualiza el DOM para que coincida con la salida del renderizado de Clock.
Cuando la salida de Clock se inserta en el DOM, React invoca al método de ciclo de vida componentDidMount(). Dentro de él, el componente Clock le pide al navegador que configure un temporizador para invocar al método tick() del componente una vez por segundo.
Cada segundo el navegador invoca al método tick(). Dentro de él, el componente Clock planifica una actualización de la interfaz de usuario al invocar a setState() con un objeto que contiene la hora actual. Gracias a la invocación a setState(), React sabe que el estado cambió e invoca de nuevo al método render() para saber qué debe estar en la pantalla. Esta vez, this.state.date en el método render() será diferente, por lo que el resultado del renderizado incluirá la hora actualizada. Conforme a eso React actualiza el DOM.
Si el componente Clock se elimina en algún momento del DOM, React invoca al método de ciclo de vida componentWillUnmount(), por lo que el temporizador se detiene.

Usar el estado correctamente
Hay tres cosas que debes saber sobre setState().

No modifiques el estado directamente
Por ejemplo, esto no volverá a renderizar un componente:

// Incorrecto
this.state.comment = 'Hello';
En su lugar utiliza setState():

// Correcto
this.setState({comment: 'Hello'});
El único lugar donde puedes asignar this.state es el constructor
Las actualizaciones del estado pueden ser asíncronas
React puede agrupar varias invocaciones a setState() en una sola actualización para mejorar el rendimiento.

Debido a que this.props y this.state pueden actualizarse de forma asincrónica, no debes confiar en sus valores para calcular el siguiente estado.

Por ejemplo, este código puede fallar en actualizar el contador:

// Incorrecto
this.setState({
  counter: this.state.counter + this.props.increment,
});
Para arreglarlo, usa una segunda forma de setState() que acepta una función en lugar de un objeto. Esa función recibirá el estado previo como primer argumento, y las props en el momento en que se aplica la actualización como segundo argumento:

// Correcto
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
Anteriormente usamos una función flecha, pero se podría haber hecho igualmente con funciones comunes:

// Correcto
this.setState(function(state, props) {
  return {
    counter: state.counter + props.increment
  };
});

Las actualizaciones de estado se fusionan
Cuando invocas a setState(), React combina el objeto que proporcionaste con el estado actual.

Por ejemplo, tu estado puede contener varias variables independientes:

  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }
Luego puedes actualizarlas independientemente con invocaciones separadas a setState():

  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
La fusión es superficial, asi que this.setState({comments}) deja intacto a this.state.posts, pero reemplaza completamente this.state.comments.

Los datos fluyen hacia abajo
Ni los componentes padres o hijos pueden saber si un determinado componente tiene o no tiene estado y no les debería importar si se define como una función o una clase.

Por eso es que el estado a menudo se le denomina local o encapsulado. No es accesible desde otro componente excepto de aquel que lo posee y lo asigna.

Un componente puede elegir pasar su estado como props a sus componentes hijos:

<FormattedDate date={this.state.date} />

El componente FormattedDate recibiría date en sus props y no sabría si vino del estado de Clock, de los props de Clock, o si se escribió manualmente:

function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}