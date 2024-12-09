import streamlit as st
import numpy as np

class MacbethMethod:
    def __init__(self, criteria, weights, alternatives):
        self.criteria = criteria
        self.weights = weights
        self.alternatives = alternatives
        self.num_criteria = len(criteria)

    def calculate_scores(self, ratings):
        scores = []
        V_plus = np.max(ratings, axis=0)  # Max values for each criterion
        V_minus = np.min(ratings, axis=0)  # Min values for each criterion

        for i in range(len(self.alternatives)):
            V_i = 0  # Global score of alternative i
            for j in range(self.num_criteria):
                r_ij = ratings[i][j]
                if V_plus[j] == V_minus[j]:  # Avoid division by zero
                    V_j = 0
                else:
                    V_j = ((r_ij - V_minus[j]) / (V_plus[j] - V_minus[j])) * 100
                V_i += V_j * self.weights[j]
            scores.append((self.alternatives[i], V_i))
        return scores

# Streamlit interface
st.title("Méthode MACBETH - Analyse Multi-Critères")

# Input alternatives
alternatives = st.text_input("Entrez les alternatives (séparées par des virgules) :")
if alternatives:
    alternatives = [alt.strip() for alt in alternatives.split(",")]

# Input criteria
criteria = st.text_input("Entrez les critères (séparés par des virgules) :")
if criteria:
    criteria = [criterion.strip() for criterion in criteria.split(",")]

# Ask if criteria are qualitative or quantitative
is_qualitative = []
for crit in criteria:
    is_qual = st.radio(f"Le critère '{crit}' est-il qualitatif ?", ("Oui", "Non"), index=1)
    is_qualitative.append(is_qual == "Oui")

# Input weights for criteria
weights = []
for crit in criteria:
    weight = st.number_input(f"Poids pour {crit} (compris entre 0 et 1) :", min_value=0.0, max_value=1.0, step=0.01)
    weights.append(weight)

# Ensure the sum of weights equals 1
if sum(weights) != 1:
    st.warning("La somme des poids ne fait pas 1. Assurez-vous que la somme soit correcte.")

# Input ratings for alternatives
ratings = []
for alt in alternatives:
    alt_ratings = []
    for j, crit in enumerate(criteria):
        if is_qualitative[j]:  # Qualitative input
            value = st.selectbox(f"Évaluation pour {alt} - {crit} (qualitatif)", ["null", "very weak", "weak", "moderate", "strong", "very strong", "extreme"])
            qualitative_mapping = {"null": 0, "very weak": 15, "weak": 30, "moderate": 50, "strong": 70, "very strong": 85, "extreme": 100}
            alt_ratings.append(qualitative_mapping[value])
        else:  # Quantitative input
            value = st.number_input(f"Évaluation pour {alt} - {crit} (quantitatif)", min_value=0.0, step=0.01)
            alt_ratings.append(value)
    ratings.append(alt_ratings)

# Once all inputs are filled, calculate the results
if st.button("Calculer les résultats"):
    if len(ratings) > 0:
        macbeth = MacbethMethod(criteria, weights, alternatives)
        scores = macbeth.calculate_scores(ratings)
        results = "\n".join([f"{alt}: {score:.2f}" for alt, score in sorted(scores, key=lambda x: x[1], reverse=True)])
        st.subheader("Scores des alternatives")
        st.text(results)
