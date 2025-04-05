<script>
</script>
 
  
  <h1>Ici c'est la ou on defini a quoi ressemble la liste (voir la photo sur discord) </h1>

  <!-- Il faut que la liste aie une structure comme ceci (recursif)

  Structure des données :

Un format récursif type :
{
  id: "abc123",
  title: "Ma liste",
  items: [
    { label: "Faire les courses", checked: false },
    {
      label: "Préparer le repas",
      checked: false,
      subItems: [
        { label: "Couper les légumes", checked: true },
        { label: "Cuire le riz", checked: false }
      ]
        
    }
  ]
} 
  
-->


  
  <!-- Ajouter un élément -->
  <input type="text" placeholder="Nouvel élément" />
  <button on:click={ajouterElement}>Ajouter</button>
  
  <style>
  </style>