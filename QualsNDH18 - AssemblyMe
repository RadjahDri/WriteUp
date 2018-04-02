# NDH Qualif 2018: AssemblyMe
## Keywords
WebAssembly - Javascript - Reverse 

## Découverte du challenge
Au lancement du chall, on tombe sur une simple page web avec un formulaire composé d'un text input et un bouton de validation.

Le code source révèle l'import d'un fichier JS supplémentaire (index.js) et la routine au clic sur le bouton de validation.

```
<script src="index.js"></script>
<script>
	var button = document.getElementById('check');

	button.addEventListener('click', function(){

	u = document.getElementById("i").value;
	var a = Module.cwrap('checkAuth', 'string', ['string']);
	var b = a(u);
	document.getElementById("x").innerHTML = b;
	});
</script>
```
<center>*Scrpt JS de la page*</center>

L'appel à la fonction Module.cwrap('checkAuth', 'string', ['string']) parait directement bien pour commencer le reverse. Une rapide recherche m'oriente vers le Web Assembly. J'en ai vite fait entendu parlé mais pervertir de l'ASM avec du JS m'a toujours semblé une hérésie.

On laisse de coté les préjugés et on ouvre la doc. Youpi .....

je précise que je n'avais aucune connaissance de WebASM avant de commencer ce chall. La méthodologie peut être foireuse comme jamais. 95% de mon temps à été perdu à cause du langage, de l'environnement, du debug et non pas pour le reverse de l'algo en temps que tel.

## Résolution
Je comprend donc que le fichier JS index.js doit être l'environnement permettant de gérer le WebASM. Je retrouve la trace de la fonction checkAuth dans la ligne suivant de ce fichier:
```
var _checkAuth = Module["_checkAuth"] = function() {  return Module["asm"]["_checkAuth"].apply(null, arguments) };
```

Plus qu'à trouver où est défini cette fonction. L'onglet debug des Developper Tools Firefox me montrent une ressource en wasm:// ce qui me parait plutot pas mal.

J'y trouve un format textuel de WebASM. En me concentrant toujours sur la fonction checkAuth, je retrouve la ligne suivante:
```
(export "_checkAuth" (func $func35))
```

J'en conclu que le WebASM doit exporter des fonctions pour quelles soient utilisables dans la partie JS.

Une fois avoir compris que le WebASM se base sur une stack d'opérande et un semblant de buffer pour gérer la mémoire, il est assez facile de comprendre le fonctionnement de la fonction. Le corps de la fonction est dans les annexes.
```
checkAuth(p0):
  if(!func57(p0, 4, 1616)):
    if(!func57(p0 + 4, 4, 1638)):
      if(!func57(p0 + 8, 5, 1610)):
        if(!func57(p0 + 13, 4, 1598)):
          if(!func57(p0 + 17, 3, 1681)):
            if(!func57(p0 + 20, 9, 1654)):
              return 1690
  // Something else
```
<center>*Pseudo code de la fonction checkAuth*</center>


La fonction semble faire une vérification partie par partie du mot de passe utilisateur donné en entrée.
- p0 serait un pointeur vers l'entrée utilisateur.
- func57 effectuerai une validation d'une portion de l'entrée utilisateur.
- On remarque aussi que le deuxième paramètre de func57 est égale au décalage appliqué à l'entrée utilisateur lors du prochain appel. Cela pourrait être un nombre de bytes à valider.
- La fin de la fonction n'a pas été étudié. Il me paraissait évident que l'on cherchait à atteindre le retour au bout de la suite de if.

Entre temps Geographer (Que je remercie), a réussi à compiler un tools de décompilation vers du C. J'ai pu continuer le reverse à partir de ce résultat (Lui aussi placé dans les annexes).

Après quelques formattages du code voici l'algorithme trouvé:
```
# Compare str referenced by pInput to str referenced by pGoal.
# Stop when str pInput end (Null byte) or when maxLen char are compared
func57(pInput, pGoal, maxLen):
  while(*pInput):
    *pGoal
    maxLen -= 1
    if(*pInput != *pGoal ||
      maxLen == 0 ||
      !*pGoal):
      break
    pGoal++
    pInput++
  return *pInput - *pGoal
```
<center>*Pseudo code de la fonction func57*</center>

La fonction effectue une simple comparaison entre les maxLen premiers bytes référencés par pInput et ceux par pGoal.

Il faut maintenant trouver ce qui est pointé lors des différents appels à func57. Le WebASM se base sur un buffer servant de mémoire. Après quelques recherches, je découvre qu'il est possible d'intérragir avec ce buffer depuis le JS.

Je vous ai passé le temps que j'ai passé sur le debugger des Developper Tools vu que je n'ai pas bien réussi à le faire fonctionner efficacement. Mais vu qu'il suffit d'accèder à l'état de la mémoire à une étape donnée, ce cas là était plus simple.

Un breakpoint posé dans le code JS juste avant l'appel à la fonction de validation permet de ne même pas entrer dans le code WebASM (Ce qui est une très bonne chose). De là la ligne JS new Int8Array(Module['buffer']) permet d'obtenir le contenu de la mémoire du programme WebASM.
Pour plus de facilité, le contenu de chacune des parties du flag ont été récupérés via cette fonction:
```
function getStr(i){
  var ret = "";
  c = -1;
  while(c != 0){
    c = buff[i];
    i++;
    ret += c;
  }
  return ret;
}
```

En appelant cette fonction avec les index passés en deuxième paramètre de func57, on récupère le flag:
```
d51XPox)1S0xk5S11W_eKXK,,,xie
```

# Annexes
## Fonction de validation: checkAuth
```
(func $func35 (param $var0 i32) (result i32)
  (local $var1 i32) (local $var2 i32)
  get_global $global5
  set_local $var1
  get_global $global5
  i32.const 16
  i32.add
  set_global $global5
  get_local $var1
  set_local $var2
  get_local $var0
  i32.const 1616
  i32.const 4
  call $func57
  i32.eqz
  if
    get_local $var0
    i32.const 4
    i32.add
    i32.const 1638
    i32.const 4
    call $func57
    i32.eqz
    if
      get_local $var0
      i32.const 8
      i32.add
      i32.const 1610
      i32.const 5
      call $func57
      i32.eqz
      if
        get_local $var0
        i32.const 13
        i32.add
        i32.const 1598
        i32.const 4
        call $func57
        i32.eqz
        if
          get_local $var0
          i32.const 17
          i32.add
          i32.const 1681
          i32.const 3
          call $func57
          i32.eqz
          if
            get_local $var0
            i32.const 20
            i32.add
            i32.const 1654
            i32.const 9
            call $func57
            i32.eqz
            if
              get_local $var1
              set_global $global5
              i32.const 1690
              return
            end
          end
        end
      end
    end
  end
  i32.const 1396
  get_local $var0
  get_local $var0
  call $func45
  call $func57
  i32.eqz
  if
    get_local $var1
    set_global $global5
    i32.const 1761
    return
  end
  i32.const 1747
  get_local $var2
  call $func77
  drop
  get_local $var1
  set_global $global5
  i32.const 1761
)
```

## Fonction de validation de partie du mot de passe: func57
```
static u32 f57(u32 p0, u32 p1, u32 p2) {
  u32 l0 = 0, l1 = 0;
  FUNC_PROLOGUE;
  u32 i0, i1, i2, i3;
  i0 = p2;
  if (i0) {
    i0 = p0;
    i0 = i32_load8_s(Z_envZ_memory, (u64)(i0));
    l0 = i0;			l0 = currentChar
    if (i0) {
      i0 = p0;
      l1 = i0;
      i0 = l0;
      p0 = i0;
      L3:
        i0 = p0;
        i1 = 24u;
        i0 <<= (i1 & 31);
        i1 = 24u;
        i0 = (u32)((s32)i0 >> (i1 & 31));
        i1 = p1;
        i1 = i32_load8_s(Z_envZ_memory, (u64)(i1));
        l0 = i1;
        i0 = i0 == i1;
        i1 = p2;
        i2 = 4294967295u;
        i1 += i2;
        p2 = i1;
        i2 = 0u;
        i1 = i1 != i2;
        i2 = l0;
        i3 = 0u;
        i2 = i2 != i3;
        i1 &= i2;
        i0 &= i1;
        i0 = !(i0);
        if (i0) {goto B1;}
        i0 = p1;
        i1 = 1u;
        i0 += i1;
        p1 = i0;
        i0 = l1;
        i1 = 1u;
        i0 += i1;
        l1 = i0;
        i0 = i32_load8_s(Z_envZ_memory, (u64)(i0));
        p0 = i0;
        if (i0) {goto L3;}
        i0 = 0u;
        p0 = i0;
    } else {
      i0 = 0u;
      p0 = i0;
    }
    B1:;
    i0 = p0;
    i1 = 255u;
    i0 &= i1;
    i1 = p1;
    i1 = i32_load8_u(Z_envZ_memory, (u64)(i1));
    i0 -= i1;
  } else {
    i0 = 0u;
  }
  p0 = i0;
  FUNC_EPILOGUE;
  return i0;
}
```
