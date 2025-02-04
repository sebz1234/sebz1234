- 👋 Hi, I’m @sebz1234
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calcul Optimisé du Nombre d'UP avec Visualisation à l'Échelle</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        .container {
            max-width: 600px;
            margin: 0 auto;
        }
        h1 {
            text-align: center;
        }
        label {
            display: block;
            margin: 10px 0 5px;
        }
        input {
            width: 100%;
            padding: 8px;
            margin-bottom: 10px;
        }
        .result {
            background-color: #f0f8ff;
            padding: 10px;
            margin-top: 20px;
            border: 1px solid #ccc;
        }
        button {
            padding: 10px 15px;
            background-color: #007bff;
            color: white;
            border: none;
            cursor: pointer;
        }
        button:hover {
            background-color: #0056b3;
        }
        canvas {
            border: 1px solid black;
            display: block;
            margin: 20px auto;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Calcul Optimisé du Nombre d'UP avec Visualisation à l'Échelle</h1>
        
        <label for="largeurFeuille">Largeur de la feuille (pouces) :</label>
        <input type="number" id="largeurFeuille" step="0.01">

        <label for="hauteurFeuille">Hauteur de la feuille (pouces) :</label>
        <input type="number" id="hauteurFeuille" step="0.01">

        <label for="largeurImpression">Largeur de l'impression (pouces) :</label>
        <input type="number" id="largeurImpression" step="0.01">

        <label for="hauteurImpression">Hauteur de l'impression (pouces) :</label>
        <input type="number" id="hauteurImpression" step="0.01">

        <label for="marge">Marge entre les impressions (pouces) :</label>
        <input type="number" id="marge" step="0.01" value="0">

        <label for="bordure">Bordure autour de la feuille (pouces) :</label>
        <input type="number" id="bordure" step="0.01" value="0.56">

        <label for="brosse">Sens du bross&eacute; &agrave; respecter :</label>
        <select id="brosse">
            <option value="oui">Oui</option>
            <option value="non">Non</option>
        </select>

        <button onclick="calculerUP()">Calculer</button>

        <div class="result" id="resultat">
            <strong>R&eacute;sultats :</strong>
            <p id="sansRotation"></p>
            <p id="avecRotation"></p>
            <p id="optimal"></p>
        </div>

        <canvas id="nestingCanvas" width="800" height="600"></canvas>
    </div>

    <script>
        function calculerUP() {
            const largeurFeuille = parseFloat(document.getElementById('largeurFeuille').value);
            const hauteurFeuille = parseFloat(document.getElementById('hauteurFeuille').value);
            const largeurImpression = parseFloat(document.getElementById('largeurImpression').value);
            const hauteurImpression = parseFloat(document.getElementById('hauteurImpression').value);
            const marge = parseFloat(document.getElementById('marge').value);
            const bordure = parseFloat(document.getElementById('bordure').value);
            const brosseRespectee = document.getElementById('brosse').value.toLowerCase() === 'oui';

            // Calcul des dimensions utilisables
            const largeurUtilisable = largeurFeuille - 2 * bordure;
            const hauteurUtilisable = hauteurFeuille - 2 * bordure;

            // Calcul des impressions possibles sans rotation
            const impressionsSansRotation = Math.floor(largeurUtilisable / (largeurImpression + marge)) *
                                             Math.floor(hauteurUtilisable / (hauteurImpression + marge));

            // Calcul des impressions possibles avec rotation
            const impressionsAvecRotation = brosseRespectee ? 0 : (Math.floor(largeurUtilisable / (hauteurImpression + marge)) *
                                                                  Math.floor(hauteurUtilisable / (largeurImpression + marge)));

            // Optimisation des UP en testant différentes orientations (maximisation)
            const maxUP = Math.max(impressionsSansRotation, impressionsAvecRotation);

            // Affichage des résultats
            document.getElementById('sansRotation').innerText = `Nombre d'impressions par feuille (sans rotation, sens respecté) : ${impressionsSansRotation}`;
            document.getElementById('avecRotation').innerText = `Nombre d'impressions par feuille (avec rotation, si autorisé) : ${impressionsAvecRotation}`;
            document.getElementById('optimal').innerText = `Nombre d'impressions optimal : ${maxUP}`;

            // Dessin du montage optimisé à l'échelle réelle
            dessinerNesting(largeurUtilisable, hauteurUtilisable, largeurImpression, hauteurImpression, marge, maxUP, maxUP === impressionsAvecRotation, bordure);
        }

        function dessinerNesting(largeurFeuille, hauteurFeuille, largeurImpression, hauteurImpression, marge, maxUP, rotationActive, bordure) {
            const canvas = document.getElementById('nestingCanvas');
            const ctx = canvas.getContext('2d');

            // Nettoyer le canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Échelle calculée pour ajuster la feuille entière au canvas
            const echelleLargeur = canvas.width / (largeurFeuille + 2 * bordure);
            const echelleHauteur = canvas.height / (hauteurFeuille + 2 * bordure);
            const echelle = Math.min(echelleLargeur, echelleHauteur);

            // Dessiner la feuille avec la bordure
            const xBordure = bordure * echelle;
            const yBordure = bordure * echelle;
            ctx.strokeStyle = 'black';
            ctx.strokeRect(xBordure, yBordure, largeurFeuille * echelle, hauteurFeuille * echelle);

            // Définition des dimensions en fonction de la rotation
            const largeurBloc = rotationActive ? hauteurImpression : largeurImpression;
            const hauteurBloc = rotationActive ? largeurImpression : hauteurImpression;

            // Placement optimisé des UP
            let compteUP = 0;

            const maxLignes = Math.floor(hauteurFeuille / (hauteurBloc + marge));
            const maxColonnes = Math.floor(largeurFeuille / (largeurBloc + marge));

            // Boucle pour dessiner les UP optimisés à l'échelle
            for (let i = 0; i < maxLignes; i++) {
                for (let j = 0; j < maxColonnes; j++) {
                    if (compteUP >= maxUP) break;
                    const x = xBordure + j * (largeurBloc + marge) * echelle;
                    const y = yBordure + i * (hauteurBloc + marge) * echelle;

                    // Dessiner le bloc
                    ctx.fillStyle = 'rgba(0, 128, 255, 0.5)';
                    ctx.fillRect(x, y, largeurBloc * echelle, hauteurBloc * echelle);
                    ctx.strokeRect(x, y, largeurBloc * echelle, hauteurBloc * echelle);

                    compteUP++;
                }
            }
        }
    </script>
</body>
</html>

<!---
sebz1234/sebz1234 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
