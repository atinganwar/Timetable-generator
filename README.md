import React, { useState } from "react";
import jsPDF from "jspdf";
import autoTable from 'jspdf-autotable';
import * as XLSX from 'xlsx';

const initialLectures = [
  { time: "7:50 am - 8:50 am" },
  { time: "8:50 am - 9:50 am" },
  { time: "9:50 am - 10:10 am", break: true },
  { time: "10:10 am - 11:10 am" },
  { time: "11:10 am - 12:10 pm" },
  { time: "12:10 pm - 1:10 pm" }
];

const days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];

export default function TimetableGenerator() {
  const [collegeName, setCollegeName] = useState("");
  const [academicYear, setAcademicYear] = useState("");
  const [facultyName, setFacultyName] = useState("");
  const [abbreviation, setAbbreviation] = useState("");
  const [timetable, setTimetable] = useState(
    initialLectures.map(() => Array(days.length).fill(""))
  );

  const handleChange = (dayIndex, lectureIndex, value) => {
    const updated = timetable.map(row => [...row]);
    if (updated[lectureIndex][dayIndex] && updated[lectureIndex][dayIndex] !== value) {
      alert("Conflict detected: Another lecture is already scheduled at this time.");
    }
    updated[lectureIndex][dayIndex] = value;
    setTimetable(updated);
  };

  const generatePDF = () => {
    const doc = new jsPDF();
    const logo = new Image();
    logo.src = '/logo.png'; // Ensure the logo is placed in the public folder

    logo.onload = () => {
      doc.addImage(logo, 'PNG', 80, 10, 50, 20); // Centered logo
      doc.setFontSize(16);
      doc.text(collegeName, 105, 40, { align: "center" }); // Below logo
      doc.setFontSize(12);
      doc.text(`Academic Year: ${academicYear}`, 105, 48, { align: "center" });
      doc.text(`Faculty/Classroom: ${facultyName}`, 105, 56, { align: "center" });

      const head = [["Time", ...days]];
      const body = timetable.map((row, i) => [initialLectures[i].time, ...row]);

      autoTable(doc, {
        startY: 65,
        head,
        body,
      });

      if (doc.autoTable.previous) {
        doc.text(`\nReport: Faculty - ${facultyName} (${abbreviation})`, 14, doc.autoTable.previous.finalY + 20);
        doc.text("\nPrincipal's Signature: ______________________", 14, doc.autoTable.previous.finalY + 30);
      }

      doc.save("timetable.pdf");
    };
  };

  const downloadExcel = () => {
    const ws = XLSX.utils.aoa_to_sheet([
      [collegeName],
      ["Academic Year: ", academicYear],
      ["Faculty/Classroom: ", facultyName],
      [""],
      ["Time", ...days],
      ...timetable.map((row, i) => [initialLectures[i].time, ...row]),
      [""],
      ["Faculty Report: ", `${facultyName} (${abbreviation})`],
      ["Principal's Signature"]
    ]);
    const wb = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb, ws, "Timetable");
    XLSX.writeFile(wb, "timetable.xlsx");
  };

  return (
    <div className="p-6 max-w-7xl mx-auto">
      <h1 className="text-3xl font-bold text-white bg-indigo-700 p-4 rounded-2xl shadow mb-6 text-center">
        Customizable College Timetable Generator
      </h1>

      <div className="grid grid-cols-2 gap-4 mb-6">
        <input
          type="text"
          placeholder="College Name"
          value={collegeName}
          onChange={(e) => setCollegeName(e.target.value)}
          className="p-2 border rounded"
        />
        <input
          type="text"
          placeholder="Academic Year"
          value={academicYear}
          onChange={(e) => setAcademicYear(e.target.value)}
          className="p-2 border rounded"
        />
        <input
          type="text"
          placeholder="Faculty/Classroom Name"
          value={facultyName}
          onChange={(e) => setFacultyName(e.target.value)}
          className="p-2 border rounded"
        />
        <input
          type="text"
          placeholder="Faculty Abbreviation"
          value={abbreviation}
          onChange={(e) => setAbbreviation(e.target.value)}
          className="p-2 border rounded"
        />
      </div>

      <table className="table-auto w-full border border-gray-400 text-center">
        <thead>
          <tr className="bg-gray-200">
            <th className="border px-4 py-2">Time</th>
            {days.map((day, idx) => (
              <th key={idx} className="border px-4 py-2">{day}</th>
            ))}
          </tr>
        </thead>
        <tbody>
          {initialLectures.map((lec, lecIdx) => (
            <tr key={lecIdx} className={lec.break ? "bg-yellow-100" : ""}>
              <td className="border px-2 py-2 font-semibold">{lec.time}</td>
              {days.map((_, dayIdx) => (
                <td key={dayIdx} className="border px-2 py-2">
                  {lec.break ? "Break" : (
                    <input
                      type="text"
                      value={timetable[lecIdx][dayIdx]}
                      onChange={(e) => handleChange(dayIdx, lecIdx, e.target.value)}
                      className="p-1 w-full border rounded"
                    />
                  )}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>

      <div className="mt-6 flex gap-4">
        <button onClick={generatePDF} className="bg-blue-600 text-white px-4 py-2 rounded">
          Download PDF
        </button>
        <button onClick={downloadExcel} className="bg-green-600 text-white px-4 py-2 rounded">
          Download Excel
        </button>
      </div>

      <div className="mt-8 bg-gray-100 p-4 rounded shadow">
        <h2 className="text-xl font-semibold mb-2">Report Summary</h2>
        <p><strong>Faculty:</strong> {facultyName}</p>
        <p><strong>Abbreviation:</strong> {abbreviation}</p>
      </div>
    </div>
  );
}
