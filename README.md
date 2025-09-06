
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BlackBird - Terminal Interface</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@300;400;500;700&display=swap');
        
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Fira Code', 'Courier New', monospace;
            background: #000000;
            color: #00ff41;
            overflow-x: hidden;
            min-height: 100vh;
            position: relative;
            cursor: crosshair;
        }

        .scanlines {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: repeating-linear-gradient(
                0deg,
                transparent,
                transparent 2px,
                rgba(0, 255, 65, 0.03) 2px,
                rgba(0, 255, 65, 0.03) 4px
            );
            pointer-events: none;
            z-index: 1000;
            animation: scanlineFlicker 0.1s infinite;
        }

        .terminal-noise {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-image: 
                radial-gradient(circle, rgba(0,255,65,0.05) 1px, transparent 1px);
            background-size: 2px 2px;
            animation: noise 0.2s infinite;
            pointer-events: none;
            z-index: 999;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            position: relative;
            z-index: 10;
        }

        .terminal-header {
            background: #111111;
            border: 2px solid #00ff41;
            border-radius: 5px;
            margin-bottom: 20px;
            box-shadow: 0 0 20px rgba(0, 255, 65, 0.3);
        }

        .terminal-bar {
            background: #333333;
            padding: 10px;
            display: flex;
            align-items: center;
            border-bottom: 1px solid #00ff41;
        }

        .terminal-buttons {
            display: flex;
            gap: 8px;
        }

        .terminal-button {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            border: 1px solid #00ff41;
        }

        .close { background: #ff0000; box-shadow: 0 0 5px #ff0000; }
        .minimize { background: #ffff00; box-shadow: 0 0 5px #ffff00; }
        .maximize { background: #00ff41; box-shadow: 0 0 5px #00ff41; }

        .terminal-title {
            margin-left: 20px;
            font-size: 0.9rem;
            color: #00ff41;
            text-shadow: 0 0 10px #00ff41;
        }

        .terminal-content {
            padding: 20px;
            background: #000000;
            min-height: 500px;
        }

        .ascii-art {
            font-family: 'Fira Code', monospace;
            font-size: 0.7rem;
            line-height: 1.1;
            color: #00ff41;
            text-align: center;
            margin-bottom: 30px;
            text-shadow: 0 0 10px #00ff41;
            animation: glitchText 2s infinite;
        }

        .prompt {
            display: flex;
            align-items: center;
            margin-bottom: 15px;
            animation: fadeInTerminal 0.5s ease-out forwards;
            opacity: 0;
        }

        .prompt-symbol {
            color: #ff0080;
            margin-right: 10px;
            font-weight: bold;
            text-shadow: 0 0 5px #ff0080;
        }

        .command-line {
            color: #00ff41;
            font-size: 1rem;
            position: relative;
        }

        .typing-cursor {
            display: inline-block;
            width: 10px;
            height: 1.2em;
            background: #00ff41;
            margin-left: 2px;
            animation: blink 1s infinite;
            box-shadow: 0 0 5px #00ff41;
        }

        .output-section {
            margin: 30px 0;
            border-left: 3px solid #ff0080;
            padding-left: 15px;
        }

        .info-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
            gap: 20px;
            margin: 30px 0;
        }

        .info-block {
            background: rgba(0, 255, 65, 0.05);
            border: 1px solid #00ff41;
            border-radius: 5px;
            padding: 20px;
            position: relative;
            overflow: hidden;
            animation: blockGlow 3s ease-in-out infinite;
        }

        .info-block::before {
            content: '';
            position: absolute;
            top: -2px;
            left: -100%;
            width: 100%;
            height: calc(100% + 4px);
            background: linear-gradient(90deg, transparent, rgba(0, 255, 65, 0.4), transparent);
            transition: left 0.6s ease;
        }

        .info-block:hover::before {
            left: 100%;
        }

        .info-block:hover {
            box-shadow: 0 0 30px rgba(0, 255, 65, 0.5);
            transform: scale(1.02);
        }

        .block-title {
            color: #ff0080;
            font-size: 1.1rem;
            font-weight: bold;
            margin-bottom: 10px;
            text-shadow: 0 0 5px #ff0080;
        }

        .block-content {
            color: #00ff41;
            font-size: 0.9rem;
            line-height: 1.6;
        }

        .skills-matrix {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
            gap: 15px;
            margin: 30px 0;
        }

        .skill-item {
            background: #111111;
            border: 1px solid #00ff41;
            padding: 15px;
            text-align: center;
            position: relative;
            transition: all 0.3s ease;
            overflow: hidden;
        }

        .skill-item::after {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255, 0, 128, 0.3), transparent);
            transition: left 0.4s ease;
        }

        .skill-item:hover::after {
            left: 100%;
        }

        .skill-item:hover {
            color: #ff0080;
            border-color: #ff0080;
            box-shadow: 0 0 15px rgba(255, 0, 128, 0.5);
            transform: translateY(-3px);
        }

        .projects-terminal {
            background: rgba(0, 0, 0, 0.8);
            border: 2px solid #ff0080;
            border-radius: 5px;
            margin: 30px 0;
            overflow: hidden;
        }

        .project-header {
            background: #333333;
            padding: 10px 15px;
            color: #ff0080;
            font-weight: bold;
            text-shadow: 0 0 5px #ff0080;
        }

        .project-list {
            padding: 20px;
        }

        .project-entry {
            margin-bottom: 20px;
            padding: 15px;
            background: rgba(0, 255, 65, 0.05);
            border-left: 3px solid #00ff41;
            transition: all 0.3s ease;
            cursor: pointer;
        }

        .project-entry:hover {
            background: rgba(255, 0, 128, 0.1);
            border-left-color: #ff0080;
            transform: translateX(10px);
            box-shadow: 0 5px 15px rgba(255, 0, 128, 0.3);
        }

        .project-name {
            color: #ff0080;
            font-weight: bold;
            font-size: 1rem;
            margin-bottom: 8px;
            text-shadow: 0 0 5px #ff0080;
        }

        .project-desc {
            color: #00ff41;
            font-size: 0.85rem;
            line-height: 1.5;
            margin-bottom: 10px;
        }

        .project-tags {
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
        }

        .tag {
            background: rgba(255, 0, 128, 0.2);
            color: #ff0080;
            padding: 3px 8px;
            border: 1px solid #ff0080;
            font-size: 0.7rem;
            border-radius: 3px;
            text-shadow: 0 0 3px #ff0080;
        }

        .status-bar {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            background: #111111;
            border-top: 1px solid #00ff41;
            padding: 10px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            font-size: 0.8rem;
            z-index: 100;
        }

        .status-left {
            display: flex;
            gap: 20px;
            align-items: center;
        }

        .status-item {
            display: flex;
            align-items: center;
            gap: 5px;
        }

        .status-indicator {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background: #00ff41;
            box-shadow: 0 0 5px #00ff41;
            animation: pulse 2s infinite;
        }

        .contact-terminal {
            background: rgba(255, 0, 128, 0.1);
            border: 2px solid #ff0080;
            border-radius: 5px;
            padding: 20px;
            margin: 30px 0;
            text-align: center;
        }

        .contact-links {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-top: 20px;
            flex-wrap: wrap;
        }

        .contact-link {
            color: #00ff41;
            text-decoration: none;
            padding: 10px 20px;
            border: 1px solid #00ff41;
            border-radius: 3px;
            transition: all 0.3s ease;
            background: rgba(0, 255, 65, 0.1);
            position: relative;
            overflow: hidden;
        }

        .contact-link::before {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(0, 255, 65, 0.3), transparent);
            transition: left 0.4s ease;
        }

        .contact-link:hover::before {
            left: 100%;
        }

        .contact-link:hover {
            color: #ff0080;
            border-color: #ff0080;
            box-shadow: 0 0 15px rgba(255, 0, 128, 0.5);
            text-shadow: 0 0 5px #ff0080;
        }

        @keyframes scanlineFlicker {
            0% { opacity: 1; }
            95% { opacity: 1; }
            96% { opacity: 0.8; }
            97% { opacity: 1; }
            98% { opacity: 0.9; }
            99% { opacity: 1; }
            100% { opacity: 1; }
        }

        @keyframes noise {
            0% { transform: translate(0, 0); }
            10% { transform: translate(-1px, -1px); }
            20% { transform: translate(1px, -1px); }
            30% { transform: translate(-1px, 1px); }
            40% { transform: translate(1px, 1px); }
            50% { transform: translate(-1px, -1px); }
            60% { transform: translate(1px, -1px); }
            70% { transform: translate(-1px, 1px); }
            80% { transform: translate(1px, 1px); }
            90% { transform: translate(-1px, -1px); }
            100% { transform: translate(0, 0); }
        }

        @keyframes glitchText {
            0%, 90%, 100% { 
                text-shadow: 0 0 10px #00ff41;
                transform: translate(0);
            }
            91% { 
                text-shadow: 2px 0 #ff0080, -2px 0 #00ff41;
                transform: translate(-1px, 0);
            }
            93% { 
                text-shadow: -2px 0 #ff0080, 2px 0 #00ff41;
                transform: translate(1px, 0);
            }
            95% { 
                text-shadow: 0 0 10px #00ff41;
                transform: translate(0);
            }
        }

        @keyframes fadeInTerminal {
            from {
                opacity: 0;
                transform: translateY(20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        @keyframes blink {
            0%, 50% { opacity: 1; }
            51%, 100% { opacity: 0; }
        }

        @keyframes blockGlow {
            0%, 100% { box-shadow: 0 0 5px rgba(0, 255, 65, 0.3); }
            50% { box-shadow: 0 0 20px rgba(0, 255, 65, 0.6); }
        }

        @keyframes pulse {
            0%, 100% { opacity: 1; transform: scale(1); }
            50% { opacity: 0.7; transform: scale(1.2); }
        }

        @media (max-width: 768px) {
            .info-grid {
                grid-template-columns: 1fr;
            }
            
            .skills-matrix {
                grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
            }
            
            .contact-links {
                flex-direction: column;
                align-items: center;
            }
            
            .status-bar {
                flex-direction: column;
                gap: 10px;
                padding: 15px;
            }
            
            .ascii-art {
                font-size: 0.5rem;
            }
        }
    </style>
</head>
<body>
    <div class="scanlines"></div>
    <div class="terminal-noise"></div>
    
    <div class="container">
        <div class="terminal-header">
            <div class="terminal-bar">
                <div class="terminal-buttons">
                    <div class="terminal-button close"></div>
                    <div class="terminal-button minimize"></div>
                    <div class="terminal-button maximize"></div>
                </div>
                <div class="terminal-title">blackbird@cybersec:~$ active_session</div>
            </div>
            
            <div class="terminal-content">
                <div class="ascii-art">
██████╗ ██╗      █████╗  ██████╗██╗  ██╗██████╗ ██╗██████╗ ██████╗ 
██╔══██╗██║     ██╔══██╗██╔════╝██║ ██╔╝██╔══██╗██║██╔══██╗██╔══██╗
██████╔╝██║     ███████║██║     █████╔╝ ██████╔╝██║██████╔╝██║  ██║
██╔══██╗██║     ██╔══██║██║     ██╔═██╗ ██╔══██╗██║██╔══██╗██║  ██║
██████╔╝███████╗██║  ██║╚██████╗██║  ██╗██████╔╝██║██║  ██║██████╔╝
╚═════╝ ╚══════╝╚═╝  ╚═╝ ╚═════╝╚═╝  ╚═╝╚═════╝ ╚═╝╚═╝  ╚═╝╚═════╝
                </div>

                <div class="prompt" style="animation-delay: 0.5s;">
                    <span class="prompt-symbol">root@blackbird:~#</span>
                    <span class="command-line">whoami</span>
                </div>

                <div class="output-section">
                    <div class="prompt" style="animation-delay: 1s;">
                        <span class="prompt-symbol">></span>
                        <span class="command-line">Bug Hunter | Reverse Engineer | 24/7 Debugger</span>
                    </div>
                    <div class="prompt" style="animation-delay: 1.5s;">
                        <span class="prompt-symbol">></span>
                        <span class="command-line">Specializing in: Cybersecurity & AI Security Research</span>
                    </div>
                    <div class="prompt" style="animation-delay: 2s;">
                        <span class="prompt-symbol">></span>
                        <span class="command-line">Status: ALWAYS_ONLINE | Coffee: ∞ | Sleep: 404_NOT_FOUND</span>
                    </div>
                </div>

                <div class="prompt" style="animation-delay: 2.5s;">
                    <span class="prompt-symbol">blackbird@terminal:~#</span>
                    <span class="command-line">cat /proc/skills</span>
                </div>

                <div class="skills-matrix">
                    <div class="skill-item">PENETRATION_TESTING</div>
                    <div class="skill-item">REVERSE_ENGINEERING</div>
                    <div class="skill-item">BUG_BOUNTY_HUNTING</div>
                    <div class="skill-item">PYTHON_EXPLOITATION</div>
                    <div class="skill-item">WEB_SECURITY_AUDIT</div>
                    <div class="skill-item">AI_MODEL_TESTING</div>
                    <div class="skill-item">CLOUD_SEC_RESEARCH</div>
                    <div class="skill-item">MALWARE_ANALYSIS</div>
                    <div class="skill-item">CRYPTOGRAPHY_IMPL</div>
                    <div class="skill-item">ZERO_DAY_HUNTING</div>
                    <div class="skill-item">SOCIAL_ENGINEERING</div>
                    <div class="skill-item">FORENSICS_ANALYSIS</div>
                </div>

                <div class="prompt" style="animation-delay: 3s;">
                    <span class="prompt-symbol">blackbird@terminal:~#</span>
                    <span class="command-line">ls -la /home/projects/active/</span>
                </div>

                <div class="projects-terminal">
                    <div class="project-header">ACTIVE_RESEARCH_PROJECTS.log</div>
                    <div class="project-list">
                        <div class="project-entry">
                            <div class="project-name">[CRITICAL] GPT_Model_Security_Suite</div>
                            <div class="project-desc">Advanced adversarial testing framework for LLM vulnerabilities. Currently exploiting prompt injection vectors and bias detection algorithms.</div>
                            <div class="project-tags">
                                <span class="tag">PYTHON</span>
                                <span class="tag">AI_SECURITY</span>
                                <span class="tag">EXPLOITATION</span>
                                <span class="tag">RESEARCH</span>
                            </div>
                        </div>

                        <div class="project-entry">
                            <div class="project-name">[HIGH] AES-256_Encrypted_Payload_Delivery</div>
                            <div class="project-desc">Military-grade encryption system with JWT authentication bypass techniques. Implementing zero-knowledge proof protocols.</div>
                            <div class="project-tags">
                                <span class="tag">CRYPTOGRAPHY</span>
                                <span class="tag">JWT_BYPASS</span>
                                <span class="tag">ZERO_DAY</span>
                            </div>
                        </div>

                        <div class="project-entry">
                            <div class="project-name">[MEDIUM] NLP_Chatbot_Exploitation_Framework</div>
                            <div class="project-desc">Automated vulnerability scanner for conversational AI systems. Real-time injection testing and privilege escalation vectors.</div>
                            <div class="project-tags">
                                <span class="tag">NLP_EXPLOIT</span>
                                <span class="tag">AUTOMATION</span>
                                <span class="tag">PRIVESC</span>
                            </div>
                        </div>

                        <div class="project-entry">
                            <div class="project-name">[HIGH] Video_AI_Backdoor_Analysis</div>
                            <div class="project-desc">Deep learning model forensics for detecting hidden backdoors in video processing AI. Reverse engineering neural network architectures.</div>
                            <div class="project-tags">
                                <span class="tag">AI_FORENSICS</span>
                                <span class="tag">BACKDOOR_DETECT</span>
                                <span class="tag">NEURAL_ANALYSIS</span>
                            </div>
                        </div>

                        <div class="project-entry">
                            <div class="project-name">[ONGOING] Bug_Bounty_Arsenal</div>
                            <div class="project-desc">Personal collection of 0-day exploits and vulnerability research. Active hunting on enterprise targets with responsible disclosure.</div>
                            <div class="project-tags">
                                <span class="tag">0DAY_RESEARCH</span>
                                <span class="tag">ENTERPRISE_TARGETS</span>
                                <span class="tag">RESPONSIBLE_DISCLOSURE</span>
                            </div>
                        </div>

                        <div class="project-entry">
                            <div class="project-name">[CLASSIFIED] Advanced_Persistent_Debugging</div>
                            <div class="project-desc">24/7 continuous security monitoring and threat hunting operations. Real-time analysis of global attack vectors.</div>
                            <div class="project-tags">
                                <span class="tag">24/7_MONITORING</span>
                                <span class="tag">THREAT_HUNTING</span>
                                <span class="tag">APT_RESEARCH</span>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="info-grid">
                    <div class="info-block">
                        <div class="block-title">CURRENT_FOCUS.exe</div>
                        <div class="block-content">
                            • Advanced AI Security Research<br>
                            • Zero-day vulnerability discovery<br>
                            • Cloud infrastructure penetration<br>
                            • Reverse engineering malware samples<br>
                            • Building custom exploitation frameworks
                        </div>
                    </div>

                    <div class="info-block">
                        <div class="block-title">SYSTEM_STATUS.log</div>
                        <div class="block-content">
                            • Uptime: 99.99% (Sleep is overrated)<br>
                            • Coffee_Level: MAXIMUM<br>
                            • Bug_Reports_Filed: ∞<br>
                            • CVEs_Discovered: [CLASSIFIED]<br>
                            • Code_Lines_Debugged: 10^6+
                        </div>
                    </div>

                    <div class="info-block">
                        <div class="block-title">ARSENAL.txt</div>
                        <div class="block-content">
                            • Custom Python exploitation scripts<br>
                            • Advanced debuggers and disassemblers<br>
                            • Machine learning attack frameworks<br>
                            • Network penetration tools<br>
                            • Social engineering methodologies
                        </div>
                    </div>

                    <div class="info-block">
                        <div class="block-title">ACHIEVEMENTS.dat</div>
                        <div class="block-content">
                            • 30+ Critical vulnerabilities discovered<br>
                            • Multiple bug bounty hall of fame entries<br>
                            • Advanced persistent threat research<br>
                            • Open source security tool contributions<br>
                            • Continuous security research publication
                        </div>
                    </div>
                </div>

                <div class="prompt" style="animation-delay: 4s;">
                    <span class="prompt-symbol">blackbird@terminal:~#</span>
                    <span class="command-line">connect --secure-channel</span>
                </div>

                <div class="contact-terminal">
                    <div class="block-title">SECURE_COMMUNICATION_CHANNELS</div>
                    <div class="block-content" style="margin-bottom: 20px;">
                        Encrypted communication available • PGP verified • Tor-friendly<br>
                        Available for: Security Research • Bug Bounty Collaboration • Penetration Testing
                    </div>
                    <div class="contact-links">
                        <a href="mailto:baveshchowdary1@gmail.com" class="contact-link">SECURE_EMAIL</a>
                        <a href="https://github.com/ubvc04" class="contact-link" target="_blank">CODE_REPOSITORY</a>
                        <a href="https://bavesh.onrender.com/" class="contact-link" target="_blank">WEB_INTERFACE</a>
                        <a href="#" class="contact-link" target="_blank">DARKNET_CONTACT</a>
                    </div>
                </div>

                <div class="prompt" style="animation-delay: 4.5s;">
                    <span class="prompt-symbol">blackbird@terminal:~#</span>
                    <span class="command-line">while true; do hack(); debug(); coffee++; done</span>
                    <span class="typing-cursor"></span>
                </div>
            </div>
        </div>
    </div>

    <div class="status-bar">
        <div class="status-left">
            <div class="status-item">
                <div class="status-indicator"></div>
                <span>SYSTEM: ONLINE</span>
            </div>
            <div class="status-item">
                <div class="status-indicator" style="background: #ff0080; box-shadow: 0 0 5px #ff0080;"></div>
                <span>HUNTING: ACTIVE</span>
            </div>
            <div class="status-item">
                <div class="status-indicator" style="background: #ffff00; box-shadow: 0 0 5px #ffff00;"></div>
                <span>DEBUGGING: 24/7</span>
            </div>
        </div>
        <div class="status-right">
            <span>blackbird@cybersec | Session: <span id="uptime">00:00:00</span></span>
        </div>
    </div>

    <script>
        // Simulate terminal typing animation
        const prompts = document.querySelectorAll('.prompt');
        prompts.forEach((prompt, index) => {
            prompt.style.animationDelay = `${index * 0.5}s`;
        });

        // Matrix rain effect
        function createMatrixRain() {
            const chars = '01アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワヲン';
            const canvas = document.createElement('canvas');
            canvas.style.position = 'fixed';
            canvas.style.top = '0';
            canvas.style.left = '0';
            canvas.style.width = '100%';
            canvas.style.height = '100%';
            canvas.style.pointerEvents = 'none';
            canvas.style.zIndex = '1';
            canvas.style.opacity = '0.1';
            document.body.appendChild(canvas);

            const ctx = canvas.getContext('2d');
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;

            const fontSize = 14;
            const columns = canvas.width / fontSize;
            const drops = Array(Math.floor(columns)).fill(1);

            function draw() {
                ctx.fillStyle = 'rgba(0, 0, 0, 0.05)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                ctx.fillStyle = '#00ff41';
                ctx.font = `${fontSize}px monospace`;

                for (let i = 0; i < drops.length; i++) {
                    const text = chars[Math.floor(Math.random() * chars.length)];
                    ctx.fillText(text, i * fontSize, drops[i] * fontSize);

                    if (drops[i] * fontSize > canvas.height && Math.random() > 0.975) {
                        drops[i] = 0;
                    }
                    drops[i]++;
                }
            }

            setInterval(draw, 33);
        }

        createMatrixRain();

        // Uptime counter
        let startTime = Date.now();
        function updateUptime() {
            const elapsed = Date.now() - startTime;
            const hours = Math.floor(elapsed / 3600000).toString().padStart(2, '0');
            const minutes = Math.floor((elapsed % 3600000) / 60000).toString().padStart(2, '0');
