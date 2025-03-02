<script lang="ts">
    import { goto } from "$app/navigation";
    const oomf = "{oomf}";
    let numTries = 0;
    let paused = $state(false);
    let asterisk = $state("");
    let showHint = $state(false);
    let isShaking = $state(false);
    let password = "";
    function handleKeydown(e: any) {
        pauseAnimation();
        if (e.key === "Enter") {
            e.preventDefault();
            if (password === "oomf") {
                goto("/home");
            } else {
                numTries += 1;
                isShaking = true;
                setTimeout(() => {
                    isShaking = false;
                    asterisk = "";
                    password = "";
                }, 500);
                if (numTries === 3) {
                    showHint = true;
                    setTimeout(() => {
                        showHint = false;
                        numTries = 0;
                    }, 3000);
                }
            }
        } else if (e.key.length === 1 && /^[a-z0-9]+$/i.test(e.key)) {
            e.preventDefault();
            asterisk += "*";
            password += e.key;
        } else if (e.key === "Backspace") {
            e.preventDefault();
            asterisk = asterisk.slice(0, -1);
            password = password.slice(0, -1);
        }
    }
    function pauseAnimation() {
        paused = true;
        setTimeout(() => {
            paused = false;
        }, 500);
    }
    function lockFocus(e: any) {
        e.target.focus();
    }
</script>

<main class="w-full h-[100vh] flex flex-col gap-12 justify-center items-center">
    <pre class="text-white font-jetbrains text-sm select-none leading-tight">
                                        ..                                     
                                     ,o""""o                                   
                                  ,o$"     o                                   
                               ,o$$$                                           
                             ,o$$$'                                            
                           ,o$"o$'                                             
                         ,o$$"$"'                                              
                      ,o$"$o"$"'                                               
                   ,oo$"$"$"$"$$`                      ,oooo$$$$$$$$oooooo.    
                ,o$$$"$"$"$"$"$"o$`..             ,$o$"$$"$"'            `oo.o 
             ,oo$$$o"$"$"$"$  $"$$$"$`o        ,o$$"o$$$o$'                 `o 
          ,o$"$"$o$"$"$"$  $"$$o$$o $$o`o   ,$$$$$o$"$$o'                    $ 
        ,o"$$"'  `$"$o$" o$o$o"  $$$o$o$oo"$$$o$"$$"$o"'                     $ 
     ,o$"'        `"$ "$$o$$" $"$o$o$$"$o$$o$o$o"$"$"`""o                   '  
   ,o$'          o$ `"$"$o "flag{oomf}$$$o"$o$$o"$$$    `$$                     
  ,o'           (     `" o$"$o"$o$$$"$o$"$"$o$"$$"$ooo|  ``                    
 $"$             `    (   `"o$$"$o$o$$ "o$o"   $o$o"$"$    )                   
(  `                   `    `o$"$$o$" "o$$     "o /|"$o"                       
 `                           `$o$$$$"" o$      "o$\|"$o'                       
                              `$o"$"$ $ "       `"$"$o$                        
       				       "$$"$$ "oo         ,$""$                        
                               $"$o$$""o"          ,o$"$                       
                               $$"$$"$ "o           `,",                       
                     ,oo$oo$$$$$$"$o$$$ ""o                                    
                  ,o$$"o"o$o$$o$$$"$o$$oo"oo                                   
                ,$"oo"$$$$o$$$$"$$$o"o$o"o"$o o                                
               ,$$$""$$o$,      `$$$$"$$$o""$o $o                              
               $o$o$"$,          `$o$"$o$o"$$o$ $$o                            
              $$$o"o$$           ,$$$$o$$o"$"$$ $o$$oo      ,                  
              "$o$$$ $`.        ,"$$o$"o$""$$$$ `"$o$$oo    `o                 
              `$o$o$"$o$o`.  ,.$$"$o$$"$$"o$$$$   `$o$$ooo    $$ooooooo        
                `$o$"$o"$"$$"$$"$"$$o$$o"$$o"        `"$o$o            `"o     
                   `$$"$"$o$$o$"$$"$ $$$  $ "           `$"$o            `o    
                      `$$"o$o"$o"$o$ "  o $$$o            `$$"o          ,$    
                         (" ""$"""     o"" "o$o             `$$ooo     ,o$$    
                              $$"""o   (   "$o$$$"o            `$o$$$o$"$'     
                                ) ) )           )  ) )            ` "'
</pre>
    <div class="font-jetbrains flex">
        <label
            class:animate-shake={isShaking}
            class="{isShaking ? 'text-red-600' : 'text-dark-green'} mr-2"
            for="password">Password:</label
        >
        <div
            class="{isShaking
                ? 'text-red-600'
                : 'text-dark-green'} focus:outline-none active:border-none inline caret-transparent
            w-fit tracking-wide"
            id="password"
            autofocus
            bind:textContent={asterisk}
            onblur={lockFocus}
            onkeydown={handleKeydown}
            role="form"
            contenteditable="true"
        ></div>
        <span
            class="ml-1 {isShaking
                ? 'bg-red-600'
                : 'bg-dark-green'} w-[1ch] {paused
                ? 'animate-none'
                : 'animate-blink'}"
        ></span>
    </div>
    <span class="text-red-600 {showHint ? 'animate-fade-in' : 'opacity-0'}"
        >Hint: There is a flag in the ASCII</span
    >
</main>
