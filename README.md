React Bartending App

(Fix)[https://dev.to/dance2die/page-not-found-on-netlify-with-react-router-58mc]
This project utilizes the cocktailDB API: https://www.thecocktaildb.com/api.php
A general walkthrough of the React code is given below.

After the file tree directory and starting files are set up the react router is brought in to display multiple pages.
```React
import { BrowserRouter as Router, Route, Switch } from "react-router-dom";
import Home from "./pages/Home";
import About from "./pages/About";
import SingleCocktail from "./pages/SingleCocktail";
import Error from "./pages/Error";
import Navbar from "./components/Navbar";
function App() {
  return (
    <div>
      <Router>
        <Navbar />
        <Switch>
          <Route exact path="/">
            <Home/>
          </Route>
          <Route path="/about">
            <About/>
          </Route>
          <Route path="/cocktail/:id">
            <SingleCocktail />
          </Route>
          <Route path="*">
            <Error />
          </Route>
        </Switch>
      </Router>
    </div>
  );
}
```

After reacter router works and the other pages can be accessed the navbar component is worked on.
```React
import { Link } from "react-router-dom";

const Navbar = () => {
  return (
    <nav className="navbar">
      <div className="nav-center">
        <Link to="/">
          <img src={logo} alt="logo" className="logo" />
        </Link>
      </div>
      <ul className="nav-links">
        <li>
          <Link to="/">Home</Link>
        </li>
        <li>
          <Link to="/about">About</Link>
        </li>
      </ul>
    </nav>
  );
};
```

Next the About page and Error pages are dealt with starting with About.
```React
const About = () => {
  return (
    <section className="section about-section">
      <h1 className="section-title">about us</h1>
      <p>
        lorem40
      </p>
    </section>
  );
};
```

Then the error page.
```React
const Error = () => {
  return (
    <section className="section error-page">
      <div className="error-container">
        <h1>oops! It's a dead end</h1>
        <Link to="/" className="btn btn-primary">
          back home
        </Link>
      </div>
    </section>
  );
};
```

Now the home page is started which has two components, the search form and the list of drinks - each drink is represented with a card which allows navigation to a single page giving info about that specific drink. The basic structure for these components and the basic structure is set up.
```React
import CocktailList from "../components/CocktailList";
import SearchForm from "../components/SearchForm";

const Home = () => {
  return (
    <main>
      <SearchForm />
      <CocktailList />
    </main>
  );
};
```

Next the context is set up with state values, pass them down to the provider and then make them available in other components around the app. useState('a') is used to ensure some drinks get rendered to the page when first rendered.
```React
import React, { useState, useContext, useEffect } from 'react'

const url = 'https://www.thecocktaildb.com/api/json/v1/1/search.php?s='
const AppContext = React.createContext()

const AppProvider = ({ children }) => {
    const [loading, setLoading] = useState(true);
    const [searchTerm, setSearchTerm] = useState("a");
    const [cocktails, setCocktails] = useState([])
  return <AppContext.Provider value={loading, searchTerm, cocktails, setSearchTerm}>{children}</AppContext.Provider>
}
// make sure use
export const useGlobalContext = () => {
  return useContext(AppContext)
}

export { AppContext, AppProvider }
```

Now the values are made available in the other cmpnts starting with the search form.
```React
const SearchForm = () => {
  const {setSearchTerm} = useGlobalContext()
  return (
    <div>
      <h2>search form component</h2>
    </div>
  )
}
```  

And then in cocktails.
```Reactconst CocktailList = () => {
  const { cocktails, loading } = useGlobalContext();

  if (loading) {
    return <Loading />;
  }
```

If no drinks match the user search a message is displayed telling the user.
```React
const CocktailList = () => {
  const { cocktails, loading } = useGlobalContext();

  if (loading) {
    return <Loading />;
  }
  if (cocktails.length < 1) {
    return <h2 className="section-title">no drinks matched your search!</h2>;
  }
  return (
    <div>
      <h2>cocktail list component</h2>
    </div>
  );
};
```

Next the Fetch request will be done to the API using two different routes, Search cocktail by name & Lookup full cocktail details by id. This will be setup in the context.js file. fetchDrinks() will be called everytime changes are made to the user input. The property names are changed from the API.
```React
  const fetchDrinks = async () => {
    setLoading(true);
    try {
      const response = await fetch(`${url}${searchTerm}`);
      const data = await response.json();
      const { drinks } = data;
      if (drinks) {
        const newCocktails = drinks.map((item) => {
          const { idDrink, strDrink, strDrinkThumb, strAlcoholic, strGlass } =
            item;
          return {
            id: idDrink,
            name: strDrink,
            image: strDrinkThumb,
            info: strAlcoholic,
            glass: strGlass,
          };
        });
        setCocktails(newCocktails);
      } else {
        setCocktails([]);
      }
      setLoading(false);
    } catch (error) {
      console.log(error);
      setLoading(false);
    }
  };
  useEffect(() => {
    fetchDrinks();
  }, [searchTerm]);
```

At this point the cocktails array should be accessed and returned on fetch requests. Next the return of CocktailList.js is constructed.
```React
  return (
    <section className="section">
      <h2 className="section-title">cocktails</h2>
      <div className="cocktails-center">
        {cocktails.map((item) => {
          return <Cocktail key={item.id} {...item} />;
        })}
      </div>
    </section>
  );
```

This renders the cocktail components which now need to be handled inside of Cocktail.js.
```React
const Cocktail = ({ image, name, id, info, glass }) => {
  return (
    <article className="cocktail">
      <div className="img-container">
        <img src={image} alt={name} />
      </div>
      <div className="cocktail-footer">
        <h3>{name}</h3>
        <h4>{glass}</h4>
        <p>{info}</p>
      </div>
    </article>
  );
};
```

Next the dynamic link which will take client to single cocktail page is taken care of.
```React
        <h4>{glass}</h4>
        <p>{info}</p>
        <Link to={`/cocktail/${id}`} className="btn btn-primary btn-details">
          details
        </Link>
      </div>
    </article>
```

At this point the list of cocktails with images and details should be displayed as well as the button to the single cocktail page displayed and working. Once inside of the single cocktail page another fetch request will be made for that cocktails specific instructions using the 2nd url which takes in drink id's.
Before work on the single cocktail page is carried out however the search form was taken care of. 
```React
const SearchForm = () => {
  const { setSearchTerm } = useGlobalContext();
  const searchValue = React.useRef("");
  return (
    <section className="section search">
      <form className="search-form">
        <div className="form-control">
          <label htmlFor="name">search your favorite cocktail</label>
   <input type="text" id="name" ref={searchValue} onChange={searchCocktail}/>
        </div>
      </form>
    </section>
  );
};
```

As the user types into the search input setSearchTerm is invoked which invokes the useEffect that fetches the drinks depending on the user input. At this point the searchCocktail function is created.
```React
  const searchCocktail = () => {
    setSearchTerm(searchValue.current.value);
  };
```

At this point the search form should be updating with new drinks as user input is added. Next a focus is set on the input.
```React
  React.useEffect(() => {
    searchValue.current.focus();
  }, []);
```

Next, to prevent reloads on submit the prevent default is used on the form using the handleSubmit function which also has to be created.
```React
      <form className="search-form" onSubmit={handleSubmit}>
      
      
 const handleSubmit = (e) => {
    e.preventDefault();
  };
```

Next the single cocktail component must be setup to render the single cocktail details on user clicks. First testing is carried out to ensure the cocktail id is retrieved.
```React

const SingleCocktail = () => {
  const { id } = useParams();

  return (
    <div>
      <h2>{id} </h2>
    </div>
  );
};
```

The above rendersthe cocktail id number when the details button is clicked. Next the singleCocktail function is worked on.
```React
const SingleCocktail = () => {
  const { id } = useParams();
  const [loading, setLoading] = React.useState(false);
  const [cocktail, setCocktail] = React.useState(null);

  React.useEffect(() => {
    setLoading(true);
    async function getCocktail() {
      try {
        const response = await fetch(`${url}${id}`);
        const data = await response.json();
        console.log(data);
      } catch (error) {}
    }
    getCocktail();
  }, [id]);
```

At this point console log should show the drinks array containing the single drink object. Next will check if a drink id exist or not as well as handle loading during fetch requests. To do this the API data needs to first be deconstructed and made available for use.
```React
    async function getCocktail() {
      try {
        const response = await fetch(`${url}${id}`);
        const data = await response.json();
        if (data.drinks) {
          const {
            strDrink: name,
            strDrinkThumb: image,
            strAlcoholic: info,
            strCategory: category,
            strGlass: glass,
            strInstructions: instructions,
            strIngredient1,
            strIngredient2,
            strIngredient3,
            strIngredient4,
            strIngredient5,
          } = data.drinks[0];
          const ingredients = [
            strIngredient1,
            strIngredient2,
            strIngredient3,
            strIngredient4,
            strIngredient5,
          ];
          const newCocktail = {
            name,
            image,
            info,
            category,
            glass,
            instructions,
            ingredients,
          };
          setCocktail(newCocktail);
        } else {
          setCocktail(null);
        }
        setLoading(false);
      } catch (error) {
        console.log(error);
        setLoading(false);
      }
    }
    getCocktail();
  }, [id]);
```

Now that the destructuring is done the return was worked on.
```React
    getCocktail();
  }, [id]);
  if (loading) {
    return <Loading />;
  }
  if (!cocktail) {
    return <h2 className="section-title">Cocktail does not exists!</h2>;
  }

  return (
    <div>
      <h2>{id} </h2>
    </div>
  );
```

Next the properties from the object are destructured and placed inside of the return.
```React
  const { name, image, category, info, glass, instructions, ingredients } =
    cocktail;
```

Then the return was done.
```React
  const { name, image, category, info, glass, instructions, ingredients } =
    cocktail;
  return (
    <section className="section cocktail-section">
      <Link to="/" className="btn btn-primary">
        back home
      </Link>
      <h2 className="section-title">{name} </h2>
      <div className="drink">
        <img src={image} alt={name} />
        <div className="drink-info">
          <p>
            <span className="drink-data">name : </span>
            {name}
          </p>
          <p>
            <span className="drink-data">category : </span>
            {category}
          </p>
          <p>
            <span className="drink-data">info : </span>
            {info}
          </p>
          <p>
            <span className="drink-data">glass : </span>
            {glass}
          </p>
          <p>
            <span className="drink-data">instructions : </span>
            {instructions}
          </p>
        </div>
      </div>
    </section>
```

Then the ingredients array is mapped over.
```React
          <p>
            <span className="drink-data">ingredients:</span>
            {ingredients.map((item, index) => {
              return item ? <span key={index}>{item}</span> : null;
            })}
          </p>
```


Project is working - just missing useCallback which is needed to prevent an infinite loop on fetchDrinks by preventing the reset unless an actual change occurs in the search input.
```React
  const fetchDrinks = useCallback( async () => {
    setLoading(true);
    try {
      const response = await fetch(`${url}${searchTerm}`);
      const data = await response.json();
      const { drinks } = data;
      if (drinks) {
        const newCocktails = drinks.map((item) => {
          const { idDrink, strDrink, strDrinkThumb, strAlcoholic, strGlass } =
            item;
          return {
            id: idDrink,
            name: strDrink,
            image: strDrinkThumb,
            info: strAlcoholic,
            glass: strGlass,
          };
        });
        setCocktails(newCocktails);
      } else {
        setCocktails([]);
      }
      setLoading(false);
    } catch (error) {
      console.log(error);
      setLoading(false);
    }
  },[searchTerm, fetchDrinks]);
```
