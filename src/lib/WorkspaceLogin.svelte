<script>
    export let onLogin;

    let workspaceCode = '';
    let error = '';

    function handleSubmit() {
        const code = workspaceCode.trim().toUpperCase();

        if (code.length < 3) {
            error = 'El código debe tener al menos 3 caracteres';
            return;
        }

        // Guardar en localStorage
        localStorage.setItem('workspaceCode', code);
        onLogin(code);
    }

    function handleKeydown(event) {
        if (event.key === 'Enter') {
            handleSubmit();
        }
    }
</script>

<div class="container">
    <div class="login-card">
        <h1>Bienvenido</h1>
        <p class="subtitle">Ingresa tu código de acceso para ver tus listas</p>

        <div class="form">
            <input
                    type="text"
                    class="code-input"
                    bind:value={workspaceCode}
                    on:keydown={handleKeydown}
                    placeholder="Ej: CITYLAB2026"
                    maxlength="20"
                    autofocus
            />

            {#if error}
                <p class="error">{error}</p>
            {/if}

            <button class="submit-btn" on:click={handleSubmit}>
                Continuar
            </button>

            <p class="help-text">
                Si no tienes un código, usa <strong>DEFAULT</strong> o cualquier código que compartas con tu grupo
            </p>
        </div>
    </div>
</div>

<style>
    .container {
        min-height: 100vh;
        display: flex;
        align-items: center;
        justify-content: center;
        padding: 24px;
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    }

    .login-card {
        background: white;
        border-radius: 24px;
        padding: 48px 32px;
        max-width: 420px;
        width: 100%;
        box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
    }

    h1 {
        font-size: 32px;
        font-weight: 700;
        color: #1E293B;
        margin-bottom: 12px;
        text-align: center;
    }

    .subtitle {
        font-size: 16px;
        color: #64748B;
        text-align: center;
        margin-bottom: 32px;
        line-height: 1.5;
    }

    .form {
        display: flex;
        flex-direction: column;
        gap: 16px;
    }

    .code-input {
        width: 100%;
        padding: 16px 20px;
        border: 2px solid #E2E8F0;
        border-radius: 12px;
        font-size: 18px;
        font-weight: 600;
        text-align: center;
        text-transform: uppercase;
        font-family: inherit;
        transition: all 0.2s;
    }

    .code-input:focus {
        outline: none;
        border-color: #5B9FD8;
        box-shadow: 0 0 0 3px rgba(91, 159, 216, 0.1);
    }

    .code-input::placeholder {
        text-transform: none;
        font-weight: 400;
        color: #94A3B8;
    }

    .error {
        color: #EF4444;
        font-size: 14px;
        text-align: center;
        margin: 0;
    }

    .submit-btn {
        width: 100%;
        padding: 16px;
        background: linear-gradient(135deg, #5B9FD8 0%, #4A8BC2 100%);
        color: white;
        border: none;
        border-radius: 12px;
        font-size: 18px;
        font-weight: 600;
        cursor: pointer;
        transition: all 0.2s;
        font-family: inherit;
    }

    .submit-btn:active {
        transform: scale(0.98);
    }

    .help-text {
        font-size: 13px;
        color: #64748B;
        text-align: center;
        line-height: 1.5;
        margin-top: 8px;
    }

    .help-text strong {
        color: #1E293B;
        font-weight: 600;
    }
</style>