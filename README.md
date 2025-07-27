# AI-Interview-Simulator
import React, { useState, useEffect, useRef } from "react";
import RobotModel from "./RobotModel";

const animatedTypingDelay = 50; // ms delay between each character for typing animation

const robotQuestions = [
  "Can you tell me about your previous job role?",
  "What skills do you have related to this job?",
  "How many years of experience do you possess?",
  "Why do you want this position?",
  "Can you describe a challenging work situation and how you handled it?",
  "How do you keep up to date with new skills in your field?",
  "Where do you see yourself in 5 years?",
  "How well do you work under pressure?"
];

// Animated text component that simulates typing line by line
function AnimatedTyping({ lines, onComplete }) {
  const [displayedLines, setDisplayedLines] = useState([]);
  const [currentLineIndex, setCurrentLineIndex] = useState(0);
  const [currentCharIndex, setCurrentCharIndex] = useState(0);
  const [currentText, setCurrentText] = useState("");
  const typingTimeout = useRef(null);

  useEffect(() => {
    if (currentLineIndex < lines.length) {
      if (currentCharIndex < lines[currentLineIndex].length) {
        typingTimeout.current = setTimeout(() => {
          setCurrentText((prev) => prev + lines[currentLineIndex][currentCharIndex]);
          setCurrentCharIndex(currentCharIndex + 1);
        }, animatedTypingDelay);
      } else {
        // Line completed, move to next line after pause
        setDisplayedLines((prev) => [...prev, lines[currentLineIndex]]);
        setCurrentCharIndex(0);
        setCurrentText("");
        setCurrentLineIndex(currentLineIndex + 1);
      }
    } else {
      if (onComplete) onComplete();
    }
    return () => clearTimeout(typingTimeout.current);
  }, [currentCharIndex, currentLineIndex, lines, onComplete]);

  return (
    <div className="text-gray-800 font-mono whitespace-pre-wrap min-h-[140px]">
      {displayedLines.map((line, idx) => (
        <div key={idx}>{line}</div>
      ))}
      <div className="text-blue-700 font-bold">{currentText}&nbsp;</div>
    </div>
  );
}

export default function AiInterviewSimulator() {
  const [jobRole, setJobRole] = useState("");
  const [skill, setSkill] = useState("");
  const [experience, setExperience] = useState("");
  const [interviewStarted, setInterviewStarted] = useState(false);
  const [currentQuestionIndex, setCurrentQuestionIndex] = useState(0);
  const [robotMessages, setRobotMessages] = useState([]);
  const [userAnswer, setUserAnswer] = useState("");
  const [userMessages, setUserMessages] = useState([]);
  const [showTyping, setShowTyping] = useState(false);
  const [interviewComplete, setInterviewComplete] = useState(false);

  // When interview starts, intro robot message + first question anim
  useEffect(() => {
    if (interviewStarted) {
      setRobotMessages([
        `Hello! I am your AI interviewer for the position of "${jobRole}". I will ask you some questions based on your skills and experience.`,
      ]);
      setCurrentQuestionIndex(0);
      setUserMessages([]);
      setInterviewComplete(false);
    }
  }, [interviewStarted, jobRole]);

  // Handle next question sequence
  const nextQuestion = () => {
    if (currentQuestionIndex < robotQuestions.length) {
      setRobotMessages((prev) => [...prev, robotQuestions[currentQuestionIndex]]);
      setShowTyping(true);
    } else {
      setInterviewComplete(true);
      setShowTyping(false);
      setRobotMessages((prev) => [
        ...prev,
        "Thank you for attending this AI interview simulation. Best of luck!",
      ]);
    }
  };

  // When animated typing finishes, show input for answer
  const handleTypingComplete = () => {
    setShowTyping(false);
  };

  // On user submitting an answer
  const handleAnswerSubmit = (event) => {
    event.preventDefault();
    if (!userAnswer.trim()) return;
    setUserMessages((prev) => [...prev, userAnswer.trim()]);
    setUserAnswer("");
    setCurrentQuestionIndex(currentQuestionIndex + 1);
    // Robot will ask next question after a short pause
    setTimeout(() => {
      nextQuestion();
    }, 800);
  };

  // Start interview trigger
  const startInterview = () => {
    if (!jobRole || !skill || !experience) {
      alert("Please fill all the fields before starting the interview.");
      return;
    }
    setInterviewStarted(true);
    setRobotMessages([]);
    setUserMessages([]);
    setCurrentQuestionIndex(0);
    nextQuestion();
  };

  return (
    <>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Roboto+Mono&display=swap');

        * {
          box-sizing: border-box;
        }
        body {
          margin: 0;
          font-family: 'Roboto Mono', monospace;
          background: linear-gradient(135deg, #eef2f3, #8e9eab);
          height: 100vh;
          display: flex;
          justify-content: center;
          align-items: center;
        }
        .container {
          background: #ffffffdd;
          border: 4px solid #222;
          padding: 2rem;
          width: 90vw;
          max-width: 1200px;
          display: grid;
          grid-template-columns: 1fr 1fr;
          grid-gap: 2rem;
          border-radius: 12px;
          box-shadow: 0 20px 40px rgba(0,0,0,0.15);
          animation: fadeInScale 1s ease forwards;
        }
        @keyframes fadeInScale {
          0% {opacity: 0; transform: scale(0.95);}
          100% {opacity: 1; transform: scale(1);}
        }
        .input-section {
          display: flex;
          flex-direction: column;
          gap: 1rem;
          justify-content: flex-start;
        }
        label {
          font-weight: 700;
          margin-bottom: 0.25rem;
          color: #222;
        }
        input[type="text"], input[type="number"] {
          border: 3px solid #000;
          padding: 0.8rem 1rem;
          font-size: 1.1rem;
          font-weight: 700;
          font-family: 'Roboto Mono', monospace;
          border-radius: 5px;
          transition: border-color 0.3s ease;
        }
        input[type="text"]:focus, input[type="number"]:focus {
          outline: none;
          border-color: #0284c7;
          background:rgb(0, 0, 0);
        }
        button {
          margin-top: 1rem;
          background: #0284c7;
          color: white;
          border: none;
          font-weight: 700;
          font-size: 1.15rem;
          padding: 0.75rem 1.5rem;
          border-radius: 6px;
          cursor: pointer;
          transition: background 0.3s ease;
          user-select: none;
        }
        button:disabled {
          background: #999;
          cursor: not-allowed;
        }
        button:hover:not(:disabled) {
          background: #0369a1;
        }
        .robot-box, .person-box {
          border: 3px solid black;
          padding: 1rem;
          height: 320px;
          overflow-y: auto;
          border-radius: 12px;
          background: #fff;
          display: flex;
          flex-direction: column;
          font-size: 1rem;
          font-weight: 600;
        }
        .robot-box {
          position: relative;
          background: #f0f9ff;
        }
        .robot-label, .person-label {
          font-weight: 700;
          text-align: center;
          margin-bottom: 0.5rem;
          color: #0369a1;
        }
        .messages {
          flex-grow: 1;
          overflow-y: auto;
          padding-right: 0.5rem;
        }
        .message-robot {
          background: #bae6fd;
          border-radius: 15px 15px 15px 0;
          padding: 10px 15px;
          margin-bottom: 10px;
          max-width: 80%;
          align-self: flex-start;
          color: #114e8a;
        }
        .message-user {
          background: #c8f7c5;
          border-radius: 15px 15px 0 15px;
          padding: 10px 15px;
          margin-bottom: 10px;
          max-width: 80%;
          align-self: flex-end;
          color: #2b540c;
        }
        .user-input-form {
          margin-top: 0.5rem;
          display: flex;
          gap: 0.5rem;
        }
        .user-input-form textarea {
          flex-grow: 1;
          resize: none;
          border: 2px solid #0284c7;
          border-radius: 6px;
          padding: 0.5rem;
          font-family: 'Roboto Mono', monospace;
          font-weight: 600;
          font-size: 1rem;
          min-height: 50px;
        }
        .user-input-form textarea:focus {
          outline: none;
          background: #e0f2fe;
        }
        @media (max-width: 768px) {
          .container {
            grid-template-columns: 1fr;
            height: 90vh;
            overflow-y: auto;
          }
          .robot-box, .person-box {
            height: 220px;
          }
        }
      `}</style>

      <div className="container" role="main" aria-label="AI Interview Simulator interface">
        {/* Input section */}
        <section className="input-section" aria-labelledby="form-section-label">
          <h2 id="form-section-label" style={{ marginBottom: "1rem", fontWeight: "bold", fontSize: "1.25rem", color: "#222" }}>
            Enter your details
          </h2>
          <label htmlFor="jobRoleInput">Enter the job role</label>
          <input
            id="jobRoleInput"
            type="text"
            placeholder="e.g. Software Developer"
            value={jobRole}
            onChange={(e) => setJobRole(e.target.value)}
            disabled={interviewStarted}
            aria-required="true"
          />
          <label htmlFor="skillInput">Enter the skill that you have</label>
          <input
            id="skillInput"
            type="text"
            placeholder="e.g. JavaScript, React"
            value={skill}
            onChange={(e) => setSkill(e.target.value)}
            disabled={interviewStarted}
            aria-required="true"
          />
          <label htmlFor="experienceInput">Enter experience (years)</label>
          <input
            id="experienceInput"
            type="number"
            min="0"
            max="50"
            placeholder="e.g. 3"
            value={experience}
            onChange={(e) => setExperience(e.target.value)}
            disabled={interviewStarted}
            aria-required="true"
          />
          <button
            onClick={startInterview}
            disabled={interviewStarted}
            aria-disabled={interviewStarted}
            aria-label="Start Interview"
          >
            Start
          </button>
        </section>

        {/* Interview simulation boxes */}
        <section aria-live="polite" aria-atomic="true" className="robot-box" aria-label="AI Robot Interviewer">
          <div className="robot-label" aria-hidden="true">Robot</div>
          <div className="messages" tabIndex="0" aria-live="polite" aria-relevant="additions">
            {robotMessages.map((msg, idx) => (
              idx === robotMessages.length - 1 && showTyping ? (
                <AnimatedTyping
                  key={idx}
                  lines={[msg]}
                  onComplete={handleTypingComplete}
                />
              ) : (
                <div key={idx} className="message-robot">{msg}</div>
              )
            ))}
          </div>
        </section>
        

        <section className="person-box" aria-label="Real person answering AI robot questions">
          <div className="person-label" aria-hidden="true">Chat or typing Answer</div>

          <div className="messages" tabIndex="0">
            {userMessages.length === 0 && <p style={{ fontStyle: "italic", color: "#666" }}>Your answers will appear here...</p>}
            {userMessages.map((msg, idx) => (
              <div key={idx} className="message-user">{msg}</div>
            ))}
          </div>

          {!interviewComplete && !showTyping && interviewStarted && (
            <form
              className="user-input-form"
              onSubmit={handleAnswerSubmit}
              aria-label="Answer input form"
            >
              <textarea
                aria-label="Your answer"
                value={userAnswer}
                onChange={(e) => setUserAnswer(e.target.value)}
                placeholder="Type your answer here..."
                maxLength={300}
                required
              />
              <button type="submit" aria-label="Submit answer">Send</button>
            </form>
          )}

          {interviewComplete && (
            <p style={{ fontWeight: "700", marginTop: "1rem", color: "#0369a1" }}>
              Interview completed. Thank you for participating!
            </p>
          )}
          
        </section>

        <section aria-live="polite" aria-atomic="true" className="robot-box" aria-label="AI Robot Interviewer">
          <div className="robot-label" aria-hidden="true">Human</div>
          <div className="messages" tabIndex="0" aria-live="polite" aria-relevant="additions">
            {robotMessages.map((msg, idx) => (
              idx === robotMessages.length - 1 && showTyping ? (
                <AnimatedTyping
                  key={idx}
                  lines={[msg]}
                  onComplete={handleTypingComplete}
                />
              ) : (
                <div key={idx} className="message-robot">{msg}</div>
              )
            ))}
          </div>
        </section>
      </div>
    </>
  );
}

