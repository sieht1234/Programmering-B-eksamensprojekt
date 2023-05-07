<script>
    // det første vi skal gøre er at hente vores variabler
    //vi henter variableen language fra app.svelte
    export let language
    //Vi henter q fra app.svelte som er everything
    export let q
    //Vi skal importere subkomponenten Article fra Article.svelte
    import Article from "./Article.svelte";
    //vi laver en variable hvis vi får en fejl og ikke kan hente data
    let errorMessage = ''
    //vi laver et tomt array til nyheder
    let news = []
    //laver en variable til artiklerne der får klassen active nr de trykkes på og sætter den til at være falsk
    let activeArticle = false 
    
        $:console.log(language)
        $: console.log(q + 'is the query')
        //hvis variablen q ændre sig kører vi et reaktivt statement
        $: if(q.length > 1){
            //v sætter news til at være et tomt array, hvis det havde noget i sig i forevejen
                news = []
                    //starter fetch kaldet ved at hente fra API'et (NEWSAPI)
                    // API'et har nogle parametre i sit kald, dette kan være q, category, language osv. Alle queries kan findes inde på NEWSAPI's hjemmeside
                    // vi indsætter vores variable q (query = søge ord) ind i URL adressen, så vi fetcher efter det der står i q, det samme gør vi med language, hvor man veælger sprog
                    fetch(`https://newsapi.org/v2/everything?q=${q}&language=${language}&sortBy=popularity&apiKey=94189c26ebd6434da9f01da114b3e217` )
                    //når vi har spurgt efter om vi må få data fra API'et, får vi et response objekt (et OK fra serven at vi godt må få adgang)
                    .then(res => {
                     //hvis vi får et OK fra serveren skal vi returnere vores svar til JSON

                   if(res.ok){
                    //vi resturnere svaret i JSON format
                       return res.json()
                       //errorMessage bliver tom fordi vi har fået adgang
                       errorMessage = ''
                       //hvis vi ikke får et OK fra serveren kører else statementet
                    }else{
                        //så skal errorMessage være kunne ikkde hente data, som tekst
                        errorMessage = 'Kunne ikke hente data fra NEWSAPI'
                        //derudover giver vi den en fejlkode, hvis vores response status er 429, viser vi på web-appen prøv igen senere
                        errorMessage += res.status == 429 ? ' - Prøv igen senere' : ''
                        //console logger det response objekt vi får
                        console.log(res)
                        }
                   })

               .then(json => {
                   //når vi har fået et OK fra serveren, går vi tilbage til den og kalder efter dataene og får et som JSOn objekter
                   console.log(json)
                    //hvis vi får JSON skal vi tage fat i arrayets articler og sætter vores variable news til at være APi'ets artikler, så vi hurtigt og nemt kan reffere til artiklerne (hvilket er hoved "arrayet" i response objektet)
                   if(json.articles) news = json.articles 
                   })
           }
    
    </script>
    
    <main class='page'>
        <div class="searchbar">
            <!--vores searchbar hvor vi binder varioablen 'q' til select elementet så den bliver til en af valgmulighederne -->
            <select bind:value={q}>
                <option value="" disabled selected>Choose</option>
                 <option value="everything">Everything</option>
                 <option value="health">Health</option>
                 <option value="apple">Apple</option>
                 <option value="business">Business</option>
                 <option value="entertainment">Entertainment</option>
                 <option value="general">General</option>
                 <option value="science">Science</option>
                 <option value="sports">Sports</option>
                 <option value="technology">Technology</option>
                
            </select>
        </div>
        <!--Polymorfi -->
        <div class={activeArticle ? 'active results' : 'results'}>
            {#if errorMessage != ''}
                 <h2>{errorMessage}</h2>
            {/if}
    
            <!--laver et if statement i HTML når vi får nogle resultater fra fetch-->
            {#if news.length > 0}
            <!--kører et each statement og løber gennem hele news arrayet som en ny variable 'n' -->
                 {#each news as n}   
                 <!--sætter hele subkomponenten Article.svelte ind med 'n' og binder activeArticle til subkomponenten så vi kan bruge den der  -->                 
                    <Article {n} bind:activeArticle={activeArticle}/>
                {/each}
                <!--hvis vi ikke har fået resultater eller ikke har fået nogle endnu laver vi en placeholder med en loading gif og tekst -->
            {:else}
                <div class="load">
                    
                    <h2>Loader...</h2>
                    <img src="./assets/giphy.gif" alt="error: missing">
    
                </div>
                <!--slutningen af if statementet -->
            {/if}
       </div>
    </main>
    
    <style>
       main{
           display:grid;
           grid-template-rows: 10vh auto;
           align-items: flex-start;
           position: relative;
           top: 20vh;
        }
       .searchbar{
           display:grid;
           background-color:rgb(69, 154, 233);
           width:100vw;
           height:10vh;
           place-items:center;
      
        }
        .searchbar select{
            display: grid;
            place-items: center;
            margin-top: .6rem;
        }
       .results{
           display:grid;
           grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
           justify-content: center;
           align-items: center;
           grid-gap:.5rem;
           width:100vw;
           padding: .2rem;
           position:relative;
          
    
        }
        .active{
            height:70vh;
            overflow:hidden;
        }
        .load{
            justify-content: center;
            display: grid;
            align-self: center;
            position: absolute;
            top: 13vh;
            left: 40vw;
            width: 40vh;
            height: 40vh;
    
        }
        .load h2{
        margin-bottom: 2rem;
        color: rgb(215, 217, 219);
       }
       select{
        border-radius: 1rem;
        padding: .2rem .5rem ;
       }
    </style>