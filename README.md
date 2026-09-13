# CryonicMite.github.io[Simulacion_Transformadas.html](https://github.com/user-attachments/files/32167748/Simulacion_Transformadas.html)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Control PID de Servomotor DC</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --bg-color: #f8fafc;
            --surface-color: #ffffff;
            --text-main: #1e293b;
            --text-muted: #64748b;
            --primary: #2563eb;
            --primary-hover: #1d4ed8;
            --border-color: #e2e8f0;
            --chart-ref: #94a3b8;
            --chart-resp: #2563eb;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-main);
            padding: 20px;
            display: flex;
            justify-content: center;
            min-height: 100vh;
        }

        .container {
            background-color: var(--surface-color);
            border-radius: 12px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
            padding: 24px;
            width: 100%;
            max-width: 1000px;
            display: grid;
            grid-template-columns: 300px 1fr;
            gap: 24px;
        }

        @media (max-width: 768px) {
            .container {
                grid-template-columns: 1fr;
            }
        }

        .panel-title {
            font-size: 1.25rem;
            font-weight: 600;
            margin-bottom: 16px;
            color: var(--text-main);
            border-bottom: 2px solid var(--border-color);
            padding-bottom: 8px;
        }

        .controls-panel {
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        .control-group {
            display: flex;
            flex-direction: column;
            gap: 8px;
            background: var(--bg-color);
            padding: 12px;
            border-radius: 8px;
            border: 1px solid var(--border-color);
        }

        .control-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .control-label {
            font-weight: 600;
            font-size: 0.9rem;
        }

        .control-value {
            font-family: monospace;
            background: var(--surface-color);
            padding: 2px 6px;
            border-radius: 4px;
            border: 1px solid var(--border-color);
            font-size: 0.85rem;
            width: 50px;
            text-align: right;
        }

        input[type=range] {
            width: 100%;
            accent-color: var(--primary);
        }

        .math-display {
            font-family: 'Courier New', Courier, monospace;
            background: #1e1e1e;
            color: #d4d4d4;
            padding: 12px;
            border-radius: 8px;
            font-size: 0.85rem;
            line-height: 1.5;
            overflow-x: auto;
        }

        .math-keyword { color: #569cd6; }
        .math-number { color: #b5cea8; }

        .btn-reset {
            background-color: var(--primary);
            color: white;
            border: none;
            padding: 10px;
            border-radius: 6px;
            cursor: pointer;
            font-weight: 600;
            transition: background-color 0.2s;
            margin-top: 10px;
        }

        .btn-reset:hover {
            background-color: var(--primary-hover);
        }

        .viz-panel {
            display: flex;
            flex-direction: column;
            gap: 16px;
        }

        .chart-container {
            position: relative;
            height: 350px;
            width: 100%;
            background: var(--surface-color);
            border: 1px solid var(--border-color);
            border-radius: 8px;
            padding: 10px;
        }

        .metrics-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 12px;
        }

        .metric-card {
            background: var(--bg-color);
            padding: 12px;
            border-radius: 8px;
            border: 1px solid var(--border-color);
            text-align: center;
        }

        .metric-label {
            font-size: 0.75rem;
            text-transform: uppercase;
            color: var(--text-muted);
            letter-spacing: 0.5px;
            margin-bottom: 4px;
        }

        .metric-value {
            font-size: 1.25rem;
            font-weight: bold;
            color: var(--primary);
        }

        .motor-params {
            font-size: 0.8rem;
            color: var(--text-muted);
            margin-top: 10px;
        }
    </style>
</head>
<body>

<div class="container">
    <!-- Panel Izquierdo: Controles -->
    <div class="controls-panel">
        <h2 class="panel-title">Controlador PID</h2>
        
        <div class="control-group">
            <div class="control-header">
                <span class="control-label">Proporcional (Kp)</span>
                <span class="control-value" id="val-kp">1.0</span>
            </div>
            <input type="range" id="slider-kp" min="0" max="20" step="0.1" value="1.0">
            <small style="color: var(--text-muted); font-size: 0.75rem;">Fuerza inicial hacia el objetivo.</small>
        </div>

        <div class="control-group">
            <div class="control-header">
                <span class="control-label">Integral (Ki)</span>
                <span class="control-value" id="val-ki">0.0</span>
            </div>
            <input type="range" id="slider-ki" min="0" max="10" step="0.1" value="0.0">
            <small style="color: var(--text-muted); font-size: 0.75rem;">Elimina error estacionario.</small>
        </div>

        <div class="control-group">
            <div class="control-header">
                <span class="control-label">Derivativo (Kd)</span>
                <span class="control-value" id="val-kd">0.1</span>
            </div>
            <input type="range" id="slider-kd" min="0" max="2" step="0.01" value="0.1">
            <small style="color: var(--text-muted); font-size: 0.75rem;">Freno predictivo / Amortiguamiento.</small>
        </div>

        <button class="btn-reset" onclick="resetParams()">Restablecer Valores</button>

        <div class="motor-params">
            <strong>Parámetros del Motor (Fijos):</strong><br>
            J (Inercia) = 0.01 kg·m²<br>
            b (Fricción) = 0.1 N·m·s<br>
            K (Constante emf/torque) = 0.01<br>
            R (Resistencia) = 1 Ω<br>
            L (Inductancia) = 0.5 H
        </div>
    </div>

    <!-- Panel Derecho: Visualización -->
    <div class="viz-panel">
        <h2 class="panel-title">Respuesta al Escalón Unitario (Posición $\Theta$)</h2>
        
        <div class="chart-container">
            <canvas id="stepResponseChart"></canvas>
        </div>

        <div class="metrics-grid">
            <div class="metric-card">
                <div class="metric-label">Tiempo de Subida (tr)</div>
                <div class="metric-value" id="metric-tr">-- s</div>
            </div>
            <div class="metric-card">
                <div class="metric-label">Sobreimpulso (OS%)</div>
                <div class="metric-value" id="metric-os">-- %</div>
            </div>
            <div class="metric-card">
                <div class="metric-label">Error Estacionario</div>
                <div class="metric-value" id="metric-ess">--</div>
            </div>
        </div>

        <div class="math-display">
            <div><span class="math-keyword">Función de Transferencia Lazo Abierto:</span></div>
            <div>G(s) = <span class="math-number">0.01</span> / (s * (<span class="math-number">0.005</span>s² + <span class="math-number">0.06</span>s + <span class="math-number">0.1001</span>))</div>
            <br>
            <div><span class="math-keyword">Controlador:</span> C(s) = Kp + Ki/s + Kds</div>
        </div>
    </div>
</div>

<script>
    // Motor Parameters (simplified for stable simulation)
    const J = 0.01;
    const b = 0.1;
    const K = 0.01;
    const R = 1.0;
    const L = 0.5;

    // Open loop Denominator coefficients before 's' multiplication: (Ls+R)(Js+b) + K^2
    // = L*J*s^2 + (L*b + R*J)*s + (R*b + K^2)
    const a2 = L * J; // 0.005
    const a1 = L * b + R * J; // 0.06
    const a0 = R * b + K * K; // 0.1001
    const numG = K; // 0.01

    let chartInstance = null;

    function initChart() {
        const ctx = document.getElementById('stepResponseChart').getContext('2d');
        chartInstance = new Chart(ctx, {
            type: 'line',
            data: {
                labels: [],
                datasets: [
                    {
                        label: 'Referencia (Posición Deseada)',
                        data: [],
                        borderColor: '#94a3b8',
                        borderDash: [5, 5],
                        borderWidth: 2,
                        pointRadius: 0,
                        fill: false
                    },
                    {
                        label: 'Posición Real θ(t)',
                        data: [],
                        borderColor: '#2563eb',
                        backgroundColor: 'rgba(37, 99, 235, 0.1)',
                        borderWidth: 2.5,
                        pointRadius: 0,
                        fill: true,
                        tension: 0.1
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                animation: {
                    duration: 0 // Disable animation for real-time slider feel
                },
                scales: {
                    x: {
                        title: { display: true, text: 'Tiempo (s)' },
                        grid: { color: '#e2e8f0' }
                    },
                    y: {
                        title: { display: true, text: 'Posición Angular (rad)' },
                        grid: { color: '#e2e8f0' },
                        suggestedMin: 0,
                        suggestedMax: 1.5
                    }
                },
                plugins: {
                    legend: { position: 'top' },
                    tooltip: { enabled: false }
                }
            }
        });
    }

    // Runge-Kutta 4th Order numerical integration for Step Response
    function simulateClosedLoop(Kp, Ki, Kd) {
        const dt = 0.01;
        const tMax = 5.0; // 5 seconds simulation
        
        let t = [];
        let y = []; // Position output
        let ref = [];

        // State Space approximation for PID + Motor 
        // 3rd order system due to Integrator in PID and 2nd order Motor dynamics
        // States:
        // x1 = position (theta)
        // x2 = velocity (omega)
        // x3 = current (i)
        // x4 = integral of error

        let x1 = 0, x2 = 0, x3 = 0, x4 = 0;
        let prevError = 0;

        let maxVal = 0;
        let finalVal = 0;
        let riseTime = -1;

        for (let i = 0; i <= tMax / dt; i++) {
            let currentTime = i * dt;
            t.push(currentTime.toFixed(2));
            ref.push(1.0); // Step input r(t) = 1

            let error = 1.0 - x1;
            
            // PID Calculation
            x4 += error * dt; // Integral of error
            let derivative = (error - prevError) / dt;
            let voltage = (Kp * error) + (Ki * x4) + (Kd * derivative);
            
            // Limit voltage (Saturation simulating real world limits)
            voltage = Math.max(Math.min(voltage, 50), -50); 
            
            prevError = error;

            // Motor dynamics derivatives
            // dx1 = x2
            // dx2 = (K*x3 - b*x2) / J
            // dx3 = (voltage - K*x2 - R*x3) / L
            
            // Simple Euler integration for speed (RK4 is better but Euler is sufficient for small dt here)
            let dx1 = x2;
            let dx2 = (K * x3 - b * x2) / J;
            let dx3 = (voltage - K * x2 - R * x3) / L;

            x1 += dx1 * dt;
            x2 += dx2 * dt;
            x3 += dx3 * dt;

            y.push(x1);

            // Metrics calculation
            if (x1 > maxVal) maxVal = x1;
            if (riseTime === -1 && x1 >= 0.9) riseTime = currentTime;
        }

        finalVal = y[y.length - 1];
        
        // Calculate Overshoot
        let overshoot = 0;
        if (maxVal > 1.0) {
            overshoot = ((maxVal - 1.0) / 1.0) * 100;
        }

        // Calculate Steady State Error
        let ess = Math.abs(1.0 - finalVal);

        return { t, y, ref, riseTime, overshoot, ess };
    }

    function updateSimulation() {
        const Kp = parseFloat(document.getElementById('slider-kp').value);
        const Ki = parseFloat(document.getElementById('slider-ki').value);
        const Kd = parseFloat(document.getElementById('slider-kd').value);

        document.getElementById('val-kp').textContent = Kp.toFixed(1);
        document.getElementById('val-ki').textContent = Ki.toFixed(1);
        document.getElementById('val-kd').textContent = Kd.toFixed(2);

        const simData = simulateClosedLoop(Kp, Ki, Kd);

        // Update Chart
        chartInstance.data.labels = simData.t;
        chartInstance.data.datasets[0].data = simData.ref;
        chartInstance.data.datasets[1].data = simData.y;
        
        // Adjust Y axis max if overshoot is huge
        let yMax = Math.max(...simData.y);
        chartInstance.options.scales.y.suggestedMax = Math.max(1.5, yMax * 1.1);
        
        chartInstance.update();

        // Update Metrics
        document.getElementById('metric-tr').textContent = simData.riseTime !== -1 ? `${simData.riseTime.toFixed(2)} s` : '> 5 s';
        document.getElementById('metric-os').textContent = `${simData.overshoot.toFixed(1)} %`;
        
        const essEl = document.getElementById('metric-ess');
        essEl.textContent = simData.ess.toFixed(3);
        if(simData.ess > 0.05) {
            essEl.style.color = '#ef4444'; // Red if error is large
        } else {
            essEl.style.color = '#10b981'; // Green if error is small
        }
    }

    function resetParams() {
        document.getElementById('slider-kp').value = 1.0;
        document.getElementById('slider-ki').value = 0.0;
        document.getElementById('slider-kd').value = 0.1;
        updateSimulation();
    }

    // Event Listeners
    document.getElementById('slider-kp').addEventListener('input', updateSimulation);
    document.getElementById('slider-ki').addEventListener('input', updateSimulation);
    document.getElementById('slider-kd').addEventListener('input', updateSimulation);

    // Init
    window.onload = () => {
        initChart();
        updateSimulation();
    };
</script>

</body>
</html>
