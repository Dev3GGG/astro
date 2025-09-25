<script setup lang="ts">
import { ref } from 'vue';
import CirclesBackground from "@components/CirclesBackground.vue";

const name = ref('');
const email = ref('');
const message = ref('');

const isSubmitting = ref(false);
const submissionStatus = ref<'idle' | 'success' | 'error'>('idle');
const statusMessage = ref('');

const handleSubmit = async () => {
    if (isSubmitting.value) return;

    isSubmitting.value = true;
    submissionStatus.value = 'idle';

    try {
        const response = await fetch("https://submit-form.com/DvEzaeUZL", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
                Accept: "application/json",
            },
            body: JSON.stringify({
                name: name.value,
                email: email.value,
                message: message.value,
            }),
        });

        if (response.ok) {
            submissionStatus.value = 'success';
            name.value = '';
            email.value = '';
            message.value = '';
        } else {
            throw new Error('La respuesta del servidor no fue exitosa.');
        }
    } catch (error) {
        console.error("Error al enviar el formulario:", error);
        submissionStatus.value = 'error';
        statusMessage.value = 'Hubo un problema al enviar tu mensaje. Por favor, inténtalo de nuevo más tarde.';
    } finally {
        isSubmitting.value = false;
        setTimeout(() => {
            submissionStatus.value = 'idle';
        }, 5000);
    }
};
</script>

<template>
    <section class="contact" id="contacto">
        <div class="circles-bg">
            <CirclesBackground client:only />
        </div>
        <div class="form-wrapper fade-up">
            <form @submit.prevent="handleSubmit" class="form-container">
                <label for="name" class="label">Nombre completo</label>
                <input v-model="name" type="text" id="name" name="name" placeholder="Nombre" required />

                <label for="email" class="label">Correo Electrónico</label>
                <input v-model="email" type="email" id="email" name="email" placeholder="ejemplo@correo.com" required />

                <label for="message" class="label">Descripción</label>
                <textarea v-model="message" id="message" name="message"
                    placeholder="Consulta para la creacion de un sistema..." required></textarea>

                <button type="submit" class="button" :disabled="isSubmitting">
                    {{ isSubmitting ? 'Enviando...' : 'Enviar' }}
                </button>
            </form>

            <div v-if="submissionStatus === 'success'" class="success-modal-overlay">
                <div class="success-modal-content">
                    <div class="icon-wrapper">
                        <svg class="checkmark-icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none"
                            stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round">
                            <polyline points="20 6 9 17 4 12"></polyline>
                        </svg>
                    </div>
                    <h2>¡Mensaje Enviado!</h2>
                    <p>Gracias por contactarte, te responderemos lo antes posible.</p>
                </div>
            </div>
            <div v-if="submissionStatus === 'error'" class="status-message error">
                {{ statusMessage }}
            </div>
        </div>
    </section>
</template>

<style>
.contact {
    width: 100%;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    overflow: hidden;
    isolation: isolate;
}

.circles-bg {
    width: 100%;
    height: 100vh;
    position: absolute;
    top: -30px;
    left: 0;
    z-index: 1;
}

.form-wrapper {
    width: 50%;
    background: lch(47.9% 8.61 273.31 / 0.118);
    padding: 2.5rem;
    border-radius: 20px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.2);
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
    position: relative;
    z-index: 2;
}

.form-container {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.label {
    font-size: 0.9rem;
    font-weight: 500;
    color: black;
    margin-bottom: -8px;
}

input,
textarea {
    width: 100%;
    box-sizing: border-box;
    padding: 14px;
    border: 1px solid #d1d9e6;
    border-radius: 10px;
    background-color: rgba(255, 255, 255, 0.9);
    font-size: 1rem;
    color: #333;
    transition:
        border-color 0.3s,
        box-shadow 0.3s;
}

input::placeholder,
textarea::placeholder {
    color: #999;
}

input:focus,
textarea:focus {
    outline: none;
    border-color: #6e8efb;
    box-shadow: 0 0 0 3px rgba(110, 142, 251, 0.3);
}

textarea {
    min-height: 120px;
    resize: vertical;
}

.button {
    padding: 14px 20px;
    border: none;
    border-radius: 10px;
    background: #4d78d3;
    color: white;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition:
        background-color 0.3s,
        transform 0.2s,
        box-shadow 0.3s;
    text-transform: uppercase;
    letter-spacing: 0.5px;
}

.button:hover:not(:disabled) {
    background: #3d63b4;
    box-shadow: 0 5px 15px rgba(61, 99, 180, 0.45);
    transform: translateY(-2px);
}

.button:active:not(:disabled) {
    transform: translateY(0);
    box-shadow: 0 2px 5px rgba(61, 99, 180, 0.3);
}

.button:disabled {
    background-color: #9ab3e8;
    cursor: not-allowed;
}

.status-message {
    padding: 1rem;
    border-radius: 8px;
    text-align: center;
    font-weight: 500;
    margin-top: 1rem;
    animation: fadeIn 0.5s ease-in-out;
}

.status-message.error {
    background-color: rgba(239, 68, 68, 0.2);
    color: #b91c1c;
    border: 1px solid rgba(239, 68, 68, 0.4);
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(-10px);
    }

    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.success-modal-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(255, 255, 255, 0.4);
    backdrop-filter: blur(8px);
    -webkit-backdrop-filter: blur(8px);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 10;
    border-radius: 20px;
    padding: 2rem;
    animation: fadeIn 0.4s ease-out;
}

.success-modal-content {
    background: #4d78d3;
    color: white;
    padding: 2.5rem 3rem;
    border-radius: 15px;
    text-align: center;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
    max-width: 400px;
    width: 100%;
}

.success-modal-content h2 {
    font-size: 1.6rem;
    font-weight: 600;
    margin: 0 0 0.5rem 0;
}

.success-modal-content p {
    font-size: 0.95rem;
    margin: 0;
    opacity: 0.9;
}

.icon-wrapper {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    background-color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 auto 1.5rem auto;
    transform: scale(0);
    animation: scaleUp 0.5s cubic-bezier(0.34, 1.56, 0.64, 1) 0.2s forwards;
}

.checkmark-icon {
    width: 32px;
    height: 32px;
    color: #4d78d3;
}

@keyframes scaleUp {
    from {
        transform: scale(0);
    }

    to {
        transform: scale(1);
    }
}


@media (max-width: 600px) {
    .contact {
        height: auto;
        padding: 4rem 0;
    }

    .circles-bg {
        height: 100%;
    }

    .form-wrapper {
        width: 80%;
        padding: 1.5rem;
    }

    .form-container {
        gap: 0.75rem;
    }

    .label {
        font-size: 0.8rem;
    }

    input,
    textarea {
        width: 100%;
        padding: 12px;
        font-size: 0.8rem;
    }

    button {
        padding: 10px;
        font-size: 0.8rem;
    }

    .success-modal-content {
        padding: 1.5rem;
    }

    .success-modal-content h2 {
        font-size: 1.2rem;
    }

    .success-modal-content p {
        font-size: 0.8rem;
    }
}
</style>
